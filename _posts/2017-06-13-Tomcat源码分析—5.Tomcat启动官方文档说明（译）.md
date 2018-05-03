---
layout: post
title: [Tomcat源码分析—5.Tomcat启动官方文档说明（译）]
categories: [Tomcat]
tags: [tomcat,源码分析,官方文档,启动]
id: [18931150487552]
fullview: false
---

查看源代码过程中，发现官网对这块的解释描述已经非常清晰了。如下是官网文档地址

[http://tomcat.apache.org/tomcat-8.0-doc/architecture/overview.html](http://tomcat.apache.org/tomcat-8.0-doc/architecture/overview.html)，该地址是Tomcat官网对Tomcat架构文档入口。如下是详细内容地址

Tomcat启动官方文档说明：[http://tomcat.apache.org/tomcat-8.0-doc/architecture/startup/serverStartup.txt](http://tomcat.apache.org/tomcat-8.0-doc/architecture/startup/serverStartup.txt) 

Tomcat启动时序图：[http://tomcat.apache.org/tomcat-8.0-doc/architecture/startup/serverStartup.pdf](http://tomcat.apache.org/tomcat-8.0-doc/architecture/startup/serverStartup.pdf) 

Tomcat处理请求时序图：[http://tomcat.apache.org/tomcat-8.0-doc/architecture/requestProcess.html](http://tomcat.apache.org/tomcat-8.0-doc/architecture/requestProcess.html) 

Tomcat处理请求时序图：[http://tomcat.apache.org/tomcat-8.0-doc/architecture/requestProcess/request-process.png](http://tomcat.apache.org/tomcat-8.0-doc/architecture/requestProcess/request-process.png) 

Tomcat授权时序图：[http://tomcat.apache.org/tomcat-8.0-doc/architecture/requestProcess/authentication-process.png](http://tomcat.apache.org/tomcat-8.0-doc/architecture/requestProcess/authentication-process.png)

如下是Tomcat启动官方文档说明

```java
Tomcat启动顺序
1. 通过命令行启动
Class: org.apache.catalina.startup.Bootstrap
该类主要做了如下几个事情:
    a) 设置classloaders
        commonLoader (common)-> System Loader
        sharedLoader (shared)-> commonLoader -> System Loader
        catalinaLoader(server) -> commonLoader -> System Loader
        (默认情况下sharedLoader和serverLoader就是commonLoader实例)
    b) 加载启动类(反射)
        org.apache.catalina.startup.Catalina
        setParentClassloader -> sharedLoader
        Thread.contextClassloader -> catalinaLoader
    c) Bootstrap.daemon.init() complete

2. 处理命令行参数(start, stop)
Class: org.apache.catalina.startup.Bootstrap (假设是start命令)
该类主要做了如下几个事情:
    a) Catalina.setAwait(true);
    b) Catalina.load()
        b1) initDirs() -> 设置属性值，比如
                          catalina.home
                          catalina.base == catalina.home (most cases)
        b2) initNaming
            setProperty(javax.naming.Context.INITIAL_CONTEXT_FACTORY,
                    org.apache.naming.java.javaURLContextFactory ->default)
        b3) createStartDigester()
            配置一个digester实例，去解析server.xml文件成对应实例，如下
            org.apache.catalina.core.StandardServer
            org.apache.catalina.deploy.NamingResources
             Stores naming resources in the J2EE JNDI tree（在J2EE JNDI树中存储资源命名）
            org.apache.catalina.LifecycleListener
                实现主要组件的start/stop事件
            org.apache.catalina.core.StandardService
                包含一组连接器Connector，使容器可以监听多个连接器。
            org.apache.catalina.Connector
                Connector只监听进入的请求
            digester也会创建如下规则集合实例。
                NamingRuleSet
                EngineRuleSet
                HostRuleSet
                ContextRuleSet
        b4) 加载server.xml文件，并且通过digester去解析它。digester会将server.xml对应节点自动映射成对象，此时实际的容器并没有启动。
        b5) 将System.out和System.err分配给SystemLogHandler类
        b6) Calls initialize on all components, this makes each object register itself with the
            JMX agent.
            During the process call the Connectors also initialize the adapters.
            The adapters are the components that do the request pre-processing.
            Typical adapters are HTTP1.1 (default if no protocol is specified,
            org.apache.coyote.http11.Http11NioProtocol)
            AJP1.3 for mod_jk etc.

    c) Catalina.start()
        c1) Starts the NamingContext and binds all JNDI references into it
        c2) 启动Server节点下的services：
            StandardService -> starts Engine (ContainerBase -> Realm,Cluster etc)
        c3) StandardHost (started by the service)
                Configures a ErrorReportValvem to do proper HTML output for different HTTP
                errors codes
                Starts the Valves in the pipeline (at least the ErrorReportValve)
                Configures the StandardHostValve,
                    this valves ties the Webapp Class loader to the thread context
                    it also finds the session for the request
                    and invokes the context pipeline
                Starts the HostConfig component
                    This component deploys all the webapps
                        (webapps & conf/Catalina/localhost/*.xml)
                    HostConfig will create a Digester for your context, this digester
                    will then invoke ContextConfig.start()
                        The ContextConfig.start() will process the default web.xml (conf/web.xml)
                        and then process the applications web.xml (WEB-INF/web.xml)

        c4) During the lifetime of the container (StandardEngine) there is a background thread that
            keeps checking if the context has changed. If a context changes (timestamp of war file,
            context xml file, web.xml) then a reload is issued (stop/remove/deploy/start)

    d) Tomcat接收一个HTTP端口的请求
        d1) The request is received by a separate thread which is waiting in the ThreadPoolExecutor
             class. It is waiting for a request in a regular ServerSocket.accept() method.
             When a request is received, this thread wakes up.
        d2) The ThreadPoolExecutor assigns the a TaskThread to handle the request.
            It also supplies a JMX object name to the catalina container (not used I believe)
        d3) The processor to handle the request in this case is Coyote Http11Processor,
            and the process method is invoked.
            This same processor is also continuing to check the input stream of the socket
            until the keep alive point is reached or the connection is disconnected.
        d4) The HTTP request is parsed using an internal buffer class (Http11InputBuffer)
            The buffer class parses the request line, the headers, etc and store the result in a
            Coyote request (not an HTTP request) This request contains all the HTTP info, such
            as servername, port, scheme, etc.
        d5) The processor contains a reference to an Adapter, in this case it is the
            CoyoteAdapter. Once the request has been parsed, the Http11Processor
            invokes service() on the adapter. In the service method, the Request contains a
            CoyoteRequest and CoyoteResponse (null for the first time)
            The CoyoteRequest(Response) implements HttpRequest(Response) and HttpServletRequest(Response)
            The adapter parses and associates everything with the request, cookies, the context through a
            Mapper, etc
        d6) When the parsing is finished, the CoyoteAdapter invokes its container (StandardEngine)
            and invokes the invoke(request,response) method.
            This initiates the HTTP request into the Catalina container starting at the engine level
        d7) The StandardEngine.invoke() simply invokes the container pipeline.invoke()
        d8) By default the engine only has one valve the StandardEngineValve, this valve simply
            invokes the invoke() method on the Host pipeline (StandardHost.getPipeLine())
        d9) the StandardHost has two valves by default, the StandardHostValve and the ErrorReportValve
        d10) The standard host valve associates the correct class loader with the current thread
             It also retrieves the Manager and the session associated with the request (if there is one)
             If there is a session access() is called to keep the session alive
        d11) After that the StandardHostValve invokes the pipeline on the context associated
             with the request.
        d12) The first valve that gets invoked by the Context pipeline is the FormAuthenticator
             valve. Then the StandardContextValve gets invoke.
             The StandardContextValve invokes any context listeners associated with the context.
             Next it invokes the pipeline on the Wrapper component (StandardWrapperValve)
        d13) During the invocation of the StandardWrapperValve, the JSP wrapper (Jasper) gets invoked
             This results in the actual compilation of the JSP.
             And then invokes the actual servlet.
    e) Invocation of the servlet class
```


