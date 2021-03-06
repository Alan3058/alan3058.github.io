---
layout: post
title: [Tomcat源码分析—1.tomcat入口]
categories: [Tomcat]
tags: [tomcat,源码分析,tomcat入口]
id: [18914373271552]
fullview: false
---

**注：该系列文章如未特殊注明，默认使用的是官方tomcat8.0.38版本**

tomcat是一个apache软件基金会下的核心项目，因其开源免费，是一款深受市场欢迎web服务器。没有特意去统计它的使用量，但工作四五年了，基本上都是和它打交道。为了做到知其然必先知其所以然，决定研究其源代码，剖析它的原理。


首先找到tomcat入口文件，我们知道启动tomcat是通过运行startup.sh/bat文件，停止服务是运行shutdown.sh/bat(sh是Linux操作系统下的运行的文件，bat是对应的是Windows操作系统)，故而tomcat入口脚本文件就是startup.sh文件。我们进入startup.sh源文件，找到42行和60行，发现它调用了catalina.sh文件，如下是部分源代码。

```bash
PRGDIR=`dirname "$PRG"`
EXECUTABLE=catalina.sh   //调用了catalina.sh文件

# Check that target executable exists
if $os400; then
  # -x will Only work on the os400 if the files are:
  # 1. owned by the user
  # 2. owned by the PRIMARY group of the user
  # this will not work if the user belongs in secondary groups
  eval
else
  if [ ! -x "$PRGDIR"/"$EXECUTABLE" ]; then
    echo "Cannot find $PRGDIR/$EXECUTABLE"
    echo "The file is absent or does not have execute permission"
    echo "This file is needed to run this program"
    exit 1
  fi
fi

exec "$PRGDIR"/"$EXECUTABLE" start "$@"
```

接着顺藤摸瓜找到catalina.sh源文件。在这里能找到了启动程序的命令，从433行到439行，如下

```bash
    eval $_NOHUP "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
      -Djava.endorsed.dirs="\"$JAVA_ENDORSED_DIRS\"" -classpath "\"$CLASSPATH\"" \
      -Dcatalina.base="\"$CATALINA_BASE\"" \
      -Dcatalina.home="\"$CATALINA_HOME\"" \
      -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
      org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 "&"
```

从上面可以看出，tomcat启动的Java类是Bootstrap，即入口方法为Bootstrap.main()。接着我们进入这个方法，查看其代码。

```java
    /**
     * Main method and entry point when starting Tomcat via the provided
     * scripts.
     *
     * @param args Command line arguments to be processed
     */
    public static void main(String args[]) {

        if (daemon == null) {
            // Don't set daemon until init() has completed
            Bootstrap bootstrap = new Bootstrap();
            try {
                bootstrap.init();
            } catch (Throwable t) {
                handleThrowable(t);
                t.printStackTrace();
                return;
            }
            daemon = bootstrap;
        } else {
            // When running as a service the call to stop will be on a new
            // thread so make sure the correct class loader is used to prevent
            // a range of class not found exceptions.
            Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);
        }

        try {
            String command = "start";
            if (args.length > 0) {
                command = args[args.length - 1];
            }

            if (command.equals("startd")) {
                args[args.length - 1] = "start";
                daemon.load(args);
                daemon.start();
            } else if (command.equals("stopd")) {
                args[args.length - 1] = "stop";
                daemon.stop();
            } else if (command.equals("start")) {
                daemon.setAwait(true);
                daemon.load(args);
                daemon.start();
            } else if (command.equals("stop")) {
                daemon.stopServer(args);
            } else if (command.equals("configtest")) {
                daemon.load(args);
                if (null==daemon.getServer()) {
                    System.exit(1);
                }
                System.exit(0);
            } else {
                log.warn("Bootstrap: command \"" + command + "\" does not exist.");
            }
        } catch (Throwable t) {
            // Unwrap the Exception for clearer error reporting
            if (t instanceof InvocationTargetException &&
                    t.getCause() != null) {
                t = t.getCause();
            }
            handleThrowable(t);
            t.printStackTrace();
            System.exit(1);
        }

    }
```

从上代码主要做了如下三件事。

* Bootstrap初始化init()。

* Bootstrap加载服务load()。

* Boostrap启动start()/停止stop()/退出exit()。


### 总结

通过shell启动文件的指令，可以找到Tomcat启动的Java方法入口，其实也可以通过查看进程信息做到的。


```bash
$ ps -ef| grep tomcat
```

Tomcat启动主要做了以下几个事情。

* Bootstrap初始化init()。

* Bootstrap加载服务load()。

* Boostrap启动start()/停止stop()/退出exit()。



