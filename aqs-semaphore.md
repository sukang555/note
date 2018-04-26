    1.Semaphore也是两个构造器，分别返回公平锁对象或者是非公平锁对象
    2.创建 Semaphore 实例的时候，需要一个参数 permits，这个基本上可以确定是设置给 AQS 的 state 的，
        然后每个线程调用 acquire 的时候，执行 state = state - 1，release 的时候执行 state = state + 1，
        当然，调用acquire 的时候，如果 state = 0，说明没有资源了就需要去排队，需要等待其他线程 release。



```java
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}

public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

1.acquire分析

public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

 public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
     if (Thread.interrupted())
         throw new InterruptedException();
     if (tryAcquireShared(arg) < 0)
         doAcquireSharedInterruptibly(arg);
}


protected int tryAcquireShared(int acquires) {
            for (;;) {
            //公平锁和非公平锁的区别就是先判断有没有线程在排队，如果有的话直接去排队.
            //这个方法的返回值如果小于0线程就需要排队。
                if (hasQueuedPredecessors())
                    return -1;
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
}


//线程排队方法
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        //这个ReentrantLock中已经分析过了
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
 }

















```



