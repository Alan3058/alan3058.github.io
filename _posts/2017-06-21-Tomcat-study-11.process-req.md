---
layout: post
title: [Tomcat源码分析—11.处理请求]
categories: [Tomcat]
tags: [tomcat,源码分析,处理请求]
id: [18956316311552]
fullview: false
---

Connector是一个实体类，可以处理多种协议的请求，比如AJP、HTTP。它委托ProtocolHandler接口的实现类去监听不同协议的端口，接收并解析该端口的请求。从通信协议上来分类，ProtocolHandler主要包含两大实现类：AJP和HTTP。从IO上来分类，主要包含三大实现类：IO、NIO和NIO2。如下图是该接口的实现子类。

![blob.png](/assets/resources/image/20170714/1500046617843029883.png "1500046617843029883.png")


以Http11Protocol实现子类为例。它将实际监听端口请求的职责委托给JIOEndpoint类，JIOEndpoint类又将职责委托给JIOEndpoint的Acceptor内部类。如下是它的内部类Acceptor的实现


```java
protected class Acceptor extends AbstractEndpoint.Acceptor {

        @Override
        public void run() {

            int errorDelay = 0;

            // Loop until we receive a shutdown command
            while (running) {

                // Loop if endpoint is paused
                while (paused && running) {
                    state = AcceptorState.PAUSED;
                    try {
                        Thread.sleep(50);
                    } catch (InterruptedException e) {
                        // Ignore
                    }
                }

                if (!running) {
                    break;
                }
                state = AcceptorState.RUNNING;

                try {
                    //if we have reached max connections, wait
                    countUpOrAwaitConnection();

                    Socket socket = null;
                    try {
                        // Accept the next incoming connection from the server
                        // socket
                        socket = serverSocketFactory.acceptSocket(serverSocket);
                    } catch (IOException ioe) {
                        countDownConnection();
                        // Introduce delay if necessary
                        errorDelay = handleExceptionWithDelay(errorDelay);
                        // re-throw
                        throw ioe;
                    }
                    // Successful accept, reset the error delay
                    errorDelay = 0;

                    // Configure the socket
                    if (running && !paused && setSocketOptions(socket)) {
                        // Hand this socket off to an appropriate processor
                        if (!processSocket(socket)) {
                            countDownConnection();
                            // Close socket right away
                            closeSocket(socket);
                        }
                    } else {
                        countDownConnection();
                        // Close socket right away
                        closeSocket(socket);
                    }
                } catch (IOException x) {
                    if (running) {
                        log.error(sm.getString("endpoint.accept.fail"), x);
                    }
                } catch (NullPointerException npe) {
                    if (running) {
                        log.error(sm.getString("endpoint.accept.fail"), npe);
                    }
                } catch (Throwable t) {
                    ExceptionUtils.handleThrowable(t);
                    log.error(sm.getString("endpoint.accept.fail"), t);
                }
            }
            state = AcceptorState.ENDED;
        }
    }
```

代码段 socket = serverSocketFactory.acceptSocket(serverSocket)；其实就是调用ServerSocket类的accept方法去监听对应端口。接收到请求后，开始处理Socket连接。

```java
    protected boolean processSocket(Socket socket) {
        // Process the request from this socket
        try {
            SocketWrapper<Socket> wrapper = new SocketWrapper<>(socket);
            wrapper.setKeepAliveLeft(getMaxKeepAliveRequests());
            wrapper.setSecure(isSSLEnabled());
            // During shutdown, executor may be null - avoid NPE
            if (!running) {
                return false;
            }
            getExecutor().execute(new SocketProcessor(wrapper));
        } catch (RejectedExecutionException x) {
            log.warn("Socket processing request was rejected for:"+socket,x);
            return false;
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            // This means we got an OOM or similar creating a thread, or that
            // the pool and its queue are full
            log.error(sm.getString("endpoint.process.fail"), t);
            return false;
        }
        return true;
    }
```

调用getExecutor().execute(new SocketProcessor(wrapper));即调用线程池去处理Socket请求，这是线程池会调度一个空闲的线程去处理该请求。如下图是官方提供的处理请求的时序图。

![](/assets/resources/image/20170714/1500049776206040429.png)

如上时序图可知，最终是交由Http11Processor去解析请求信息，并创建Request和Response对象。之后将Request和Response对象交由Engine引擎容器处理，Engine容器最后将请求交由过滤器链过滤处理，最后让对应Sevelet类处理并响应。

