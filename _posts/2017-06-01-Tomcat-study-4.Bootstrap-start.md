---
layout: post
title: [Tomcat源码分析—4.Bootstrap启动]
categories: [Tomcat]
tags: [tomcat,源码分析,bootstrap启动]
id: [18926956183552]
fullview: false
---

前面已经讲完了Bootstrap初始化、加载服务，接下来就是Boostrap的启动Bootstrap.start()，如下代码。

```java
    /**
     * Start the Catalina daemon.
     */
    public void start()
        throws Exception {
        if( catalinaDaemon==null ) init();

        Method method = catalinaDaemon.getClass().getMethod("start", (Class [] )null);
        method.invoke(catalinaDaemon, (Object [])null);

    }
```

从上面代码可以看出，实际上Boostrap.start()方法是通过Java反射去调用Catalina.start()方法。如下是Catalina.start()方法代码块。

```java
    /**
     * Start a new server instance.
     */
    public void start() {

        if (getServer() == null) {
            load();
        }

        if (getServer() == null) {
            log.fatal("Cannot start server. Server instance is not configured.");
            return;
        }

        long t1 = System.nanoTime();

        // Start the new server
        try {
            getServer().start();
        } catch (LifecycleException e) {
            log.fatal(sm.getString("catalina.serverStartFail"), e);
            try {
                getServer().destroy();
            } catch (LifecycleException e1) {
                log.debug("destroy() failed for failed Server ", e1);
            }
            return;
        }

        long t2 = System.nanoTime();
        if(log.isInfoEnabled()) {
            log.info("Server startup in " + ((t2 - t1) / 1000000) + " ms");
        }

        // Register shutdown hook
        if (useShutdownHook) {
            if (shutdownHook == null) {
                shutdownHook = new CatalinaShutdownHook();
            }
            Runtime.getRuntime().addShutdownHook(shutdownHook);

            // If JULI is being used, disable JULI's shutdown hook since
            // shutdown hooks run in parallel and log messages may be lost
            // if JULI's hook completes before the CatalinaShutdownHook()
            LogManager logManager = LogManager.getLogManager();
            if (logManager instanceof ClassLoaderLogManager) {
                ((ClassLoaderLogManager) logManager).setUseShutdownHook(
                        false);
            }
        }

        if (await) {
            await();
            stop();
        }
    }
```

如上，该方法主要进行了如下几个步骤。


* getServer().start()，即调用StandardServer.start()方法。

* Runtime.addShutdownHook(new CatalinaShutdownHook())，添加关闭钩子，以便服务关闭时清理释放资源。

* await(),实际是调用StandardServer.await()去创建ServerSocket，并监听端口。

* stop(),实际是调用调用StandardServer.stop()停止服务，并释放资源。



**总结**


到这里，基本上已经完成了tomcat的启动流程。如下是tomcat启动流程时序图

![mmexport1496972649225.jpg](/assets/resources/image/20170609/1496972822770016293.jpg "1496972822770016293.jpg")


