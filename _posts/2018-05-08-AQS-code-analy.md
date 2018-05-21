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
a. prev: 上一个节点  
b. next: 下一个节点  
c. waitStatus: 节点等待状态字段，用于Codition条件等待队列。只能是以下几个枚举值。  
1. SIGNAL(-1): 线程得到运行信号，可以运行。
2. CANCELLED(1): 线程已经被取消。
3. CONDITION(-2): 线程正在队列中等待。
4. PROPAGATE(-3): 线程共享释放锁，传播？？？
5. 0: 初始值

d. thread: 在这个节点上排队的线程  
e. nextWaiter:   


## ConditionObject
ConditioObject主要分两类方法，一类是await方法（await、awaitUninterruptibly、awaitNanos、awaitUntil），一类是signal方法（signal、signalAll），并且await和signal分别类同于Object的wait和notify方法，功能作用相似。  

### 属性
ConditionObject内部通过维护了一个双向等待队列Node，去实现await（线程阻塞）和signal（线程唤醒）的功能。因此它的内部包含了队列的头部和尾部Node节点两个属性字段。  
firstWaiter: 队列头部Node节点  
lastWaiter: 队列尾部Node节点  

### await阻塞方法
```java
        public final void await() throws InterruptedException {
			//被中断，则抛中断异常
            if (Thread.interrupted())
                throw new InterruptedException();
			//添加一个节点到等待队列中，并返回该节点
            Node node = addConditionWaiter();
			//释放排他锁
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            while (!isOnSyncQueue(node)) {
				//不在同步队列中，则阻塞等待
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
			//否则表示被唤醒，则尝试去获取队列同步锁
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
```

### signal唤醒方法
```java
        public final void signal() {
			//当前线程必须同时拥有排它锁，才可执行唤醒方法，否则直接异常
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            if (first != null)
				//将等待队列的首节点移动到同步队列中
                doSignal(first);
        }
```


# AQS主要方法
### acquire方法
```java
    public final void acquire(int arg) {
		//尝试获取锁；若未获取到，则往队列尾部添加一个排它锁节点，并循环队列获取节点
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
			//获取失败，中断当前线程
            selfInterrupt();
    }
```

### acquireShared方法
```java
    public final void acquireShared(int arg) {
		//尝试获取共享锁
        if (tryAcquireShared(arg) < 0)
			//获取失败，则往队列尾部添加一个共享锁，并循环获取节点
            doAcquireShared(arg);
    }
```

### release方法
```java
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
```java
   public final boolean releaseShared(int arg) {
   		//尝试释放排它锁
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```

