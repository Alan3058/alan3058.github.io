---
layout: post
title: [自动化发布-7.Gradle脚本编写]
categories: [自动化发布]
tags: [自动化发布,gradle,gradle脚本编写]
id: [18893401751552]
fullview: false
---
在前面我们使用Jenkins插件执行远程shell，其实我们可以通过不需要ssh插件，所有事情可以放在Gradle脚本中执行。这里我举个典型的案例。

通过Jenkins Gitlab插件可以实现获取代码仓库的项目源代码，并且编译项目源代码。可以在Gradle中引入ssh插件，让Gradle去执行应用重启发布。如下脚本。
```gradle
buildscript {
  repositories {
    maven {
      url "http://192.168.1.88/nexus/content/groups/public/"
    }
  }
  dependencies {
    classpath 'com.oracle:ojdbc6:11.2.0.3'
    classpath 'mysql:mysql-connector-java:5.1.40'
    classpath "org.flywaydb:flyway-gradle-plugin:4.0.3"
    classpath 'org.hidetake:gradle-ssh-plugin:2.7.0'
  }
}

apply plugin: "org.flywaydb.flyway"
apply plugin: "org.hidetake.ssh"

ext {
	projectName="test"
	tomcatHome = "/app/tomcat/apache-tomcat.8.0.59"
	appHome = "/app/application"
	env = System.getProperty("env")
}
//项目打包
task process (type:Exec) {
	workingDir '.'
	commandLine 'tar','-zcf','test.tar.gz','test'
}

//加载properties属性文件
def loadProperties(){
	def configPath = "../deploy-project-config/sale-support/config/${env}/config.properties"
    def props = new Properties()
	def configFile = file(configPath)
    configFile.withInputStream{stream -> props.load(stream)}
    props
}
//设置flyway插件连接属性
flyway {
	Properties props = loadProperties()
    user = props.'conn1.jdbc.username'
    password = props.'conn1.jdbc.password'
    url = 'jdbc:oracle:thin:@(DESCRIPTION= (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))(LOAD_BALANCE = no)(CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = xszcsrv)(FAILOVER_MODE =(TYPE = SELECT)(METHOD = BASIC)(RETRIES = 180)(DELAY = 5))))'
	locations = ['filesystem:sql']
}

//设置远程服务器地址信息
remotes {
  webServer31 {
    host = '192.168.1.31'
    user = 'test'
    password = 'password'
	//identity = file('/root/.ssh/id_rsa') 
  }
  webServer32 {
    host = '192.168.1.32'
    user = 'test'
    password = 'password'
	//identity = file('/root/.ssh/id_rsa') 
  }
  webServer33 {
    host = '192.168.1.33'
    user = 'test'
    password = 'password'
	//identity = file('/root/.ssh/id_rsa') 
  }
}
//ssh插件设置,并定义方法
ssh.settings {  
  knownHosts = allowAnyHosts
  extensions.add transfer: { 
	//传输war文件
	println "transfer war file to /tmp"
	execute "rm -rf /tmp/${projectName}"
	put from: "deploy/${projectName}.tar.gz", into: '/tmp'
	//解压war文件
	println "unzip war file"
	execute ("tar -zxf /tmp/${projectName}.tar.gz -C /tmp",ignoreError: true)
	
	Properties props = loadProperties()
	//替换properties属性
	props.each{
		execute "sed -i 's#^${it.key}=.*#${it.key}=${it.value}#g'  /tmp/${projectName}/WEB-INF/classes/config.properties"
	}
	execute "sed -i 's#uatsales.evergrandelife.com.cn:8090/hd_server/#sales.evergrandelife.com.cn/#g'  /tmp/${projectName}/hd_client_pc/app/base/main.js"
	execute "sed -i 's#/hd_server/##' /tmp/${projectName}/src-js/XConfig.js"
	execute "sed -i 's#/hd_server##' /tmp/${projectName}/src-js/XConfig.js"
	execute "sed -i 's#/hd_server/##' /tmp/${projectName}/WEB-INF/classes/client/config-runtime.xml"
	execute "sed -i 's#/hd_server##' /tmp/${projectName}/WEB-INF/classes/client/config-runtime.xml"
	execute ("mv /tmp/${projectName}/app/${env}/* /share/resource/clientSync/app/",ignoreError: true)
  }
  extensions.add backup: {
	//备份模版工程
	println "begin to backup ${projectName}"
	def date = new Date().format('yyyyMMddHHmmss')  	
	execute ("cd ${appHome} && tar -zcf backup/${projectName}${date}.tar.gz ${projectName}",ignoreError: true)
	println "backup ${projectName} completed"
  }
  extensions.add shutdown: { 			
	//关闭tomcat服务
	println "shutdown tomcat server"
	def result = execute ("source ~/.bash_profile;sh ${tomcatHome}/bin/shutdown.sh")
	
	for(i in 1..24){
		execute ("sleep 5")
		result = execute ("ps -ef |grep tomcat |grep java|grep -v grep",ignoreError: true)
		if(result.size()==0){
		  break;
		}
	}
	assert result.size()==0
  }
  extensions.add startup: {
	//移动项目文件
	println "startup ${projectName}"
	execute "rm -rf ${appHome}/${projectName}"
	execute "mv /tmp/${projectName} ${appHome}/${projectName}"
	//添加软连接
	execute "ln -s /share/resource/webRoot ${appHome}/${projectName}/webRoot"
	//启动服务
	execute "source ~/.bash_profile;sh ${tomcatHome}/bin/startup.sh"
  }
  
  extensions.add check: {
	//检查启动状态
	def result
	for(i in 1..24){
		execute ("sleep 5")
		result = execute("curl -o /dev/null -s -w %{http_code}  'http://localhost:8090' ",ignoreError: true)
		if(result=='200'||result=='302'){
		  break;
		}
	}
	assert (result=='200'||result=='302')
	
	//删除临时目录项目文件
	execute("rm -rf /tmp/${projectName} && rm -f /tmp/${projectName}.tar.gz",ignoreError: true)
	//只保留30天备份发布文件
	execute("find /app/application/backup -mtime +30 -name '*' -exec rm -rf {} \\;",ignoreError: true)
  }
}

//定义发布任务
task deployServer << {
  ssh.run {
	session(remotes.webServer31) {
		execute "curl --insecure 'https://192.168.1.31/dynamic?upstream=backend&server=${remote.host}:8090&down='"
		execute "curl --insecure 'https://192.168.1.32/dynamic?upstream=backend&server=${remote.host}:8090&down='"
		transfer()
		shutdown()
		backup()
		startup()
		check()
		execute "curl --insecure 'https://192.168.1.31/dynamic?upstream=backend&server=${remote.host}:8090&up='"
		execute "curl --insecure 'https://192.168.1.32/dynamic?upstream=backend&server=${remote.host}:8090&up='"
	}
  }
  ssh.run {
	session(remotes.webServer32,remotes.webServer33) {
		execute "curl --insecure 'https://192.168.1.31/dynamic?upstream=backend&server=${remote.host}:8090&down='"
		execute "curl --insecure 'https://192.168.1.32/dynamic?upstream=backend&server=${remote.host}:8090&down='"
		transfer()
		//针对其他机器单独关闭定时器
		execute "sed -i 's#^ePolicyBatchSendSimpleTrigger=.*#ePolicyBatchSendSimpleTrigger=0 */1 * * * ? 2025#g'  /tmp/${projectName}/WEB-INF/classes/config.properties"
		execute "sed -i 's#^simpleTriggerqili=.*#simpleTriggerqili=0 */1 * * * ? 2025#g'  /tmp/${projectName}/WEB-INF/classes/config.properties"
		execute "sed -i 's#^policyPayConvertSimpleTrigger=.*#policyPayConvertSimpleTrigger=0 */1 * * * ? 2025#g'  /tmp/${projectName}/WEB-INF/classes/config.properties"
		execute "sed -i 's#^upPortraitPdfSimpleTrigger=.*#upPortraitPdfSimpleTrigger=0 */1 * * * ? 2025#g'  /tmp/${projectName}/WEB-INF/classes/config.properties"
		//execute "sed -i '/<list>/,/<\\/list>/c <list><\\/list>' /tmp/${projectName}/WEB-INF/classes/spring/spring-config-timmer.xml"
		shutdown()
		backup()
		startup()
		check()
		execute "curl --insecure 'https://192.168.1.31/dynamic?upstream=backend&server=${remote.host}:8090&up='"
		execute "curl --insecure 'https://192.168.1.32/dynamic?upstream=backend&server=${remote.host}:8090&up='"
	}
  }
}
```

![](http://ctosb.com/ueditor/dialogs/attachment/fileTypeImages/icon_txt.gif)[deploy.txt](/assets/resources/file/20170412/1491982338115026985.txt "deploy.txt")
