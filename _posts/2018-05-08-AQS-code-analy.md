---
layout: post
title:  "AQS源码分析"
categories: [java]
tags: [java]
fullview: false
published: false
---

# AQS(AbstractQueuedSynchronizer)主要字段
head: 队列头  
tail: 队列尾  
state:  同步器状态值，实际值含义由具体同步器定义。提供了三个方法去操作这个值。getState,setState,casState

# AQS内部类
## Node  
a. prev:上一个节点  
b. next:下一个节点  
c. waitStatus:节点状态字段，只能是以下几个枚举值。
1. SIGNAL(-1): 线程得到运行信号，可以运行。
2. CANCELLED(1): 线程已经被取消。
3. CONDITION(-2): 线程正在队列中等待。
4. PROPAGATE(-3): 线程共享释放锁，传播？？？
5. 0: 初始值

## ConditionObject

# AQS主要方法
### acquire方法
```
    public final void acquire(int arg) {
		//尝试获取锁；若未获取到，则往队列尾部添加一个排它锁节点，并循环队列获取节点
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
			//获取失败，中断当前线程
            selfInterrupt();
    }
```

### acquireShared方法
```
    public final void acquireShared(int arg) {
		//尝试获取共享锁
        if (tryAcquireShared(arg) < 0)
			//获取失败，则往队列尾部添加一个共享锁，并循环获取节点
            doAcquireShared(arg);
    }
```

### release方法
```
    public final boolean release(int arg) {
		//尝试释放排它锁
        if (tryRelease(arg)) {
			//释放成功
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

### releaseShared方法
```
   public final boolean releaseShared(int arg) {
   		//尝试释放排它锁
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```

