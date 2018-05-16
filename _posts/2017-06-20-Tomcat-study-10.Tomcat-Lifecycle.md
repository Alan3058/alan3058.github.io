---
layout: post
title: [Tomcat源码分析—10.Tomcat生命周期接口Lifecycle]
categories: [Tomcat]
tags: [tomcat,源码分析,lifecycle,生命周期接口]
id: [18952122007552]
fullview: false
---

# 1.Lifecycle

Tomcat容器里有个重要的接口，管理各个组件的生命周期接口Lifecycle，查看该接口源代码，该接口主要提供初始化init()、启动start()、停止stop()、销毁destroy()、配置监听器的方法find/remove/addLifecycleListener()。Tomcat所有组件的基本上都继承了Lifecycle，也就是说Tomcat的组件都包含生命周期管理。如下图是Lifecycle接口的子类继承信息。

![blob.png](/assets/resources/image/20170623/1498190295738015419.png "1498190295738015419.png")

因此上图中描述的Tomcat组件：Server,Service,Container,Engine,Host,Context等。它们都有生命周期管理，和一个共同的状态流转关系。如下图是Lifecycle接口的描述和状态流转图。

![blob.png](/assets/resources/image/20170623/1498178691749036471.png "1498178691749036471.png")

从上图中可以看出Lifecycle主要有五个大节点：New、Init、Start、Stop、Destroy，其中有些节点被拆分成两个或三个子节点（处理前、处理中、处理完成）处理，每个节点对应一个状态。拆分后全状态：New、Initializing、Initialized、Starting_prep、Starting、Started、Stopping_prep、Stopping、Stopped、Destroying、Destroyed、Failed。

* 组件在启动过程中，任何节点都可以转换到Failed节点，然后进入Destroy节点销毁释放资源。

* 当组件状态是Starting_prep、Starting、Started，此时调用start()方法，组件不受影响。

* 当组件状态是New，此时调用start()方法，组件会立即调用init()方法，随后进入start()方法。

* 当组件状态是Stopping_prep、Stopping、Stopped，此时调用stop()方法，组件不受影响。

* 当组件状态是New，此时调用stop()方法，组件会立即转成Stopped状态。典型的案例是当一个组件启动失败并且它的所有子组件没有启动。当组件是Stopped状态时，它将会停止所有子组件，即使他们没有启动。


# 2.Lifecycle子类LifecycleBase

查看LifecycleBase源码，该类实现了父类Lifecycle所有方法。首先看init()方法。

```java
    @Override
    public final synchronized void init() throws LifecycleException {
        if (!state.equals(LifecycleState.NEW)) {
            invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
        }

        try {
            setStateInternal(LifecycleState.INITIALIZING, null, false);
            initInternal();
            setStateInternal(LifecycleState.INITIALIZED, null, false);
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            setStateInternal(LifecycleState.FAILED, null, false);
            throw new LifecycleException(
                    sm.getString("lifecycleBase.initFail",toString()), t);
        }
    }
```

如上代码，组件初始化工作主要执行了如下几个事情。

* 首先将状态设置为Initializing，并且触发before_init事件，使对应的监听器执行相应的操作。

* 然后调用initInternal()方法去进行实际的初始化工作。

* 初始化完成后在将状态设为Initialized，并且触发after_init事件，使对应的监听器执行相应的操作。

* 如果初始化过程中出现异常，则将状态置为Failed，并且触发failed事件，使对应的监听器执行相应的操作。


initInternal()是抽象方法，该方法交由子类去实现，即让子组件去实现对应的初始化工作。细心的朋友应该发现这里用到了一种设计模式：模版方法模式。

同理，start()、stop()、destroy()方法实现都和init()方法类似，都使用模版方法模式设计思想。

# 3.总结

Lifecycle是Tomcat组件生命周期管理的接口类，该接口定义了一个组件初始化init()、启动start()、停止stop()、销毁destroy()、配置监听器的方法find/remove/addLifecycleListener()等方法。LifecycleBase是Lifecycle接口类的子类，该类实现了接口所有的方法，它还定义实现了组件所有状态流转，并且将实际的初始化、启动、停止、销毁工作交由它的子类实现。


