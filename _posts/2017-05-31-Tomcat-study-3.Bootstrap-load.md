---
layout: post
title: [Tomcat源码分析—3.Bootstrap加载服务]
categories: [Tomcat]
tags: [tomcat,源码分析,bootstrap加载服务]
id: [18922761879552]
fullview: false
---

**Bootstrap加载服务**

在上篇中已经知道tomcat启动时做了三个步骤：Bootstrap初始化、Bootstrap加载服务、Bootstrap启动/停止/退出。在这里我们继续研究Bootstrap加载参数流程，看看在加载参数中做了些什么操作。进入Boostrap.load(args)方法查看，如下代码。


```bash
   /**
     * Load daemon.
     */
    private void load(String[] arguments)
        throws Exception {

        // Call the load() method
        String methodName = "load";
        Object param[];
        Class<?> paramTypes[];
        if (arguments==null || arguments.length==0) {
            paramTypes = null;
            param = null;
        } else {
            paramTypes = new Class[1];
            paramTypes[0] = arguments.getClass();
            param = new Object[1];
            param[0] = arguments;
        }
        Method method =
            catalinaDaemon.getClass().getMethod(methodName, paramTypes);
        if (log.isDebugEnabled())
            log.debug("Calling startup class " + method);
        method.invoke(catalinaDaemon, param);

    }
```

这里主要利用Java反射机制执行了Catalina.load(args)方法。接着我们找到Catalina.load(args)方法代码。


```java
    /*
     * Load using arguments
     */
    public void load(String args[]) {

        try {
            if (arguments(args)) {
                load();
            }
        } catch (Exception e) {
            e.printStackTrace(System.out);
        }
    }
```

Catalina.load(args)方法主要有两步。


* 调用arguments(args)方法，解析传入进来参数，比如入参-help，则会显示对应help信息。

* 调用load()加载tomcat配置文件server.xml信息。


如下是load()方法代码块。


```java
    /**
     * Start a new server instance.
     */
    public void load() {

        long t1 = System.nanoTime();

        initDirs();

        // Before digester - it may be needed
        initNaming();

        // Create and execute our Digester
        Digester digester = createStartDigester();

        InputSource inputSource = null;
        InputStream inputStream = null;
        File file = null;
        try {
            try {
                file = configFile();
                inputStream = new FileInputStream(file);
                inputSource = new InputSource(file.toURI().toURL().toString());
            } catch (Exception e) {
                if (log.isDebugEnabled()) {
                    log.debug(sm.getString("catalina.configFail", file), e);
                }
            }
            if (inputStream == null) {
                try {
                    inputStream = getClass().getClassLoader()
                        .getResourceAsStream(getConfigFile());
                    inputSource = new InputSource
                        (getClass().getClassLoader()
                         .getResource(getConfigFile()).toString());
                } catch (Exception e) {
                    if (log.isDebugEnabled()) {
                        log.debug(sm.getString("catalina.configFail",
                                getConfigFile()), e);
                    }
                }
            }

            // This should be included in catalina.jar
            // Alternative: don't bother with xml, just create it manually.
            if (inputStream == null) {
                try {
                    inputStream = getClass().getClassLoader()
                            .getResourceAsStream("server-embed.xml");
                    inputSource = new InputSource
                    (getClass().getClassLoader()
                            .getResource("server-embed.xml").toString());
                } catch (Exception e) {
                    if (log.isDebugEnabled()) {
                        log.debug(sm.getString("catalina.configFail",
                                "server-embed.xml"), e);
                    }
                }
            }


            if (inputStream == null || inputSource == null) {
                if  (file == null) {
                    log.warn(sm.getString("catalina.configFail",
                            getConfigFile() + "] or [server-embed.xml]"));
                } else {
                    log.warn(sm.getString("catalina.configFail",
                            file.getAbsolutePath()));
                    if (file.exists() && !file.canRead()) {
                        log.warn("Permissions incorrect, read permission is not allowed on the file.");
                    }
                }
                return;
            }

            try {
                inputSource.setByteStream(inputStream);
                digester.push(this);
                digester.parse(inputSource);
            } catch (SAXParseException spe) {
                log.warn("Catalina.start using " + getConfigFile() + ": " +
                        spe.getMessage());
                return;
            } catch (Exception e) {
                log.warn("Catalina.start using " + getConfigFile() + ": " , e);
                return;
            }
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    // Ignore
                }
            }
        }

        getServer().setCatalina(this);
        getServer().setCatalinaHome(Bootstrap.getCatalinaHomeFile());
        getServer().setCatalinaBase(Bootstrap.getCatalinaBaseFile());

        // Stream redirection
        initStreams();

        // Start the new server
        try {
            getServer().init();
        } catch (LifecycleException e) {
            if (Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE")) {
                throw new java.lang.Error(e);
            } else {
                log.error("Catalina.start", e);
            }
        }

        long t2 = System.nanoTime();
        if(log.isInfoEnabled()) {
            log.info("Initialization processed in " + ((t2 - t1) / 1000000) + " ms");
        }
    }
```

其中主要做了如下几个事情。

* initDirs()，初始化Java临时目录，校验是否有设置Java临时目录。

* initNaming(),初始化系统参数，以便后续使用。

* createStartDigester(),创建Digester实例，以便解析server.xml。

* 构建server.xml的输入流InputStream和InputSource对象。

* 将InputSource实例交给digester解析并实例化Server对象。

* getServer().init()，初始化Server实例。



**StandardServer初始化**

以下是server.xml的内容，以下每一个xml节点都对应tomcat的一个实体类。即实例化StandardServer对象时，会同时实例化Service、Connector、Engine、Realm、Listener等对象。这部分将会在后面章节详解，在这里只是为了摸清个大概脉路。


```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener"/>
  <Listener SSLEngine="on" className="org.apache.catalina.core.AprLifecycleListener"/>
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"/>
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener"/>
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener"/>

  <GlobalNamingResources>
    <Resource auth="Container" description="User database that can be updated and saved" factory="org.apache.catalina.users.MemoryUserDatabaseFactory" name="UserDatabase" pathname="conf/tomcat-users.xml" type="org.apache.catalina.UserDatabase"/>
  </GlobalNamingResources>

  <Service name="Catalina">
    <Connector connectionTimeout="20000" port="80" protocol="HTTP/1.1" redirectPort="8443"/>
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
    <Engine defaultHost="localhost" name="Catalina">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
      </Realm>

      <Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" pattern="%h %l %u %t &quot;%r&quot; %s %b" prefix="localhost_access_log" suffix=".txt"/>

      <Context docBase="template-project" path="/template-project" reloadable="true" source="org.eclipse.jst.j2ee.server:template-project"/></Host>
    </Engine>
  </Service>
</Server>
```

**总结**

Bootstrap加载参数过程中，主要是根据传入进来的参数进行启动/停止服务。如果是启动服务，则会去根据server.xml文件实例化对应的必要组件，比如Server、Service、Connector、Engine、Realm、Listener等对象，并调用Server的init()方法进行初始化操作。

