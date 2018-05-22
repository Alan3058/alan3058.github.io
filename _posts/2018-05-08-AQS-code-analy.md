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
3. CONDITION(-2): 线程正在等待队列中。
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
		//尝试获取状态；若未获取到，则往队列尾部添加一个排它锁节点，并自旋去获取状态
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
			//获取失败，中断当前线程
            selfInterrupt();
    }
```

acquireQueued主要是去自旋地获取状态
```java
    final boolean acquireQueued(final Node node, int arg) {
		//返回获取状态是否失败
        boolean failed = true;
        try {
            boolean interrupted = false;
			//自旋去获取状态
            for (;;) {
                final Node p = node.predecessor();
				//前节点是队列头节点，并且尝试获取状态
                if (p == head && tryAcquire(arg)) {
					//获取成功，将当前节点设为队列头结点
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
				//获取失败，判断是否要park。park后，再检查线程是否已中断？？？
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

### acquireShared方法
```java
    public final void acquireShared(int arg) {
		//尝试获取共享状态
        if (tryAcquireShared(arg) < 0)
			//获取失败，则往队列尾部添加一个共享节点，并自旋获取共享状态
            doAcquireShared(arg);
    }
```

doAcquireShared主要是自旋的去获取共享状态，和doAcquire方法逻辑类似
```java
    private void doAcquireShared(int arg) {
		//往队列添加一个共享节点
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
			//自旋去获取状态
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
					//尝试获取共享状态
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
						//设置队列头为当前节点
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

```

### release方法
```java
    public final boolean release(int arg) {
		//尝试释放排它锁状态
        if (tryRelease(arg)) {
			//释放成功
            Node h = head;
            if (h != null && h.waitStatus != 0)
				//唤醒头结点的下一个节点，如果下一个节点是取消状态，则跳过并唤醒下一个节点
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

### releaseShared方法
```java
   public final boolean releaseShared(int arg) {
   		//尝试释放共享锁状态
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```

```java
    private void doReleaseShared() {
        /*
         * Ensure that a release propagates, even if there are other
         * in-progress acquires/releases.  This proceeds in the usual
         * way of trying to unparkSuccessor of head if it needs
         * signal. But if it does not, status is set to PROPAGATE to
         * ensure that upon release, propagation continues.
         * Additionally, we must loop in case a new node is added
         * while we are doing this. Also, unlike other uses of
         * unparkSuccessor, we need to know if CAS to reset status
         * fails, if so rechecking.
         */
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
					//当前状态为signal，则直接设置节点状态为初始值0
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
					//唤醒下一个节点
                    unparkSuccessor(h);
                }
				//当前状态为0，则设置节点状态为传播状态
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```

