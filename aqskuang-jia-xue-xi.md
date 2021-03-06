
---

AQS类图结构

![](/assets/2.png)

ReentrantLock类图结构

![](/assets/3.png)

![](/assets/5.png)

一般我们在程序中这样使用reentrantLock对象

```java
public class AQS {

    private static final ReentrantLock reentrantLock = new ReentrantLock(true);

    public void work(){
        reentrantLock.lock();

        try {

            Thread currentThread = Thread.currentThread();
            System.out.println(currentThread.getName() +"执行任务");
            TimeUnit.SECONDS.sleep(5);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            reentrantLock.unlock();
        }
    }

}
```

```java
abstract static class Sync extends AbstractQueuedSynchronizer
```

```java
 public ReentrantLock() {
        sync = new NonfairSync();
 }


 public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
 }
```

首先当我们调用new ReentrantLock\(\)时，构造器中给我们返回了NonfairSync非公平锁对象或者是公平锁对象FairSync;

然后我们调用lock\(\)方法。接下来我们再来看lock的源码：

```java
public void lock() {
        sync.lock();
}


static final class FairSync extends Sync {

        final void lock() {
            acquire(1);
        }


        //首先进入的是这个方法，当tryAcquire 返回true的时候那么整个lock流程也就结束了。
        //假设两个线程T1和T2先后进入lock方法，t1正常情况下tryAcquire返回true，接下来我们来看tryAcquire源码。

        public final void acquire(int arg) {
            if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                selfInterrupt();
        }


}
```

```java
protected final boolean tryAcquire(int acquires) {
            //获取当前线程
            final Thread current = Thread.currentThread();

            //获取锁的state的值
            int c = getState();

            //当state==0时也就是初始化状态。
            if (c == 0) {

                //t1线程走到这里，正常情况下前一个判断返回的是false,接下来就会进行CAS对state的值进行加1操作。
                //如果CAS成功的话返回true那么FairSync的属性exclusiveOwnerThread的值就是当前获取到锁的线程,也就是t1。
                //FairSync的state的值经过CAS操作已经变为1了。现在我们来看一下这个判断条里的三个方法。

                if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }

            //如果FairSync的state不等于0的话再判断一下当前线程是否已经获取过锁了，
            //如果已经或去过那么就state+1，这也就是重入锁。返回true；

            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            //如果所有条件都不满足的话那么返回false;
            return false;
}

  public final boolean hasQueuedPredecessors() {
        //tail代表尾部节点，head代表头部节点。此时FairSync的这两个属性都为null；因此第一次返回false；
        Node t = tail; 
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
 }

  protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
 } 

 protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
 }
```

t1线程获取完锁以后接下来我们来分析t2线程，现在再次回到之前的方法。

```java
public final void acquire(int arg) {

    //t2线程此时也走到这里。同样先进入tryAcquire方法。
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}


protected final boolean tryAcquire(int acquires) {
            //同样获取当前线程 T2
            final Thread current = Thread.currentThread();
            //获取T2线程的state的值，此时获取的state的值大于0 我们之前分析过T1，
            //T1线程获取完线程以后会将state的值通过cas方式加1操作，
            //如果是重入的话进行累加操作，每获取一次就累加。所以这个方法会返回false;
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
}
        //if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))  selfInterrupt();

        //我们又回到这个方法，T2前一个判断返回的是false;那么程序会执行addWaiter()方法,
        //并将返回结果当作参数继续执行acquireQueued()方法；

private Node addWaiter(Node mode) {

        //此时new了node，mode为null 这里我们看一下这个构造器，我们把它命名为node-1
        Node node = new Node(Thread.currentThread(), mode);

        //此时mode为null，thread就是当前线程T2
            Node(Thread thread, Node mode) {     // Used by addWaiter
                    this.nextWaiter = mode;
                    this.thread = thread;
        }

        //此时的tail同样为null，那么程序不会进入这个判断  
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }

        //此时T2会进入这个方法
        enq(node);
        //这里将node-1返回，再回到之前的方法
        return node;
}

private Node enq(final Node node) {

        //此时T2进入这里，首先进入一个死循环这种方式编译出来的指令要比while少。
        for (;;) {
        //这个里 tail为null，参数为Node-1也就是T2线程的包装Node。然后会进入第一个判断，这里通过cas操作给FairSync对象的head赋值，
        //此时又new了一个空的Node作为FairSync的head节点。再将head赋值给tail节点。我们把它命名为node-0
        //然后程序又进行下一次循环，t！= null 
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
            //第二次循环会进入这个判断内，这个node就是node-1
            //然后node-1的前一个就是node-0,再次通过cas操作把tail设为node-1，node-0的下一个就是node-1，FairSync对象就有了一个链表，
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    //这里将node-1返回,再回到之前的方法
                    return t;
                }
            }
        }
}

    //T2线程将addWaiter返回的node-1传入这里，arg=1
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;

            //这里node.predecessor()将node的前一个返回,p为Node-0;

            for (;;) {
                final Node p = node.predecessor();

                //这里p==head,然后再进行一次tryAcquire操作。
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
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
    //这里第一个参数为当前节点的前一个，node为当前节点
 private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            //前驱节点的 waitStatus == -1 ，说明前驱节点状态正常，当前线程需要挂起，直接可以返回true
            return true;

            //如果前驱结点的waitStatus> 0
            //前驱节点 waitStatus大于0 ，之前说过，大于0 说明前驱节点取消了排队。这里需要知道这点：
            //进入阻塞队列排队的线程会被挂起，而唤醒的操作是由前驱节点完成的。
            //所以下面这块代码说的是将当前节点的prev指向waitStatus<=0的节点     
        if (ws > 0) {

            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            //如果前驱结点的值为0，就将前驱结点的值设置为-1;
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
}

    //由于shouldParkAfterFailedAcquire和parkAndCheckInterrupt是在一个循环中，所以第二次循环的话
    //shouldParkAfterFailedAcquire会返回true,最后挂起当前线程。
private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
}


# 二 、唤醒操作
    public void unlock() {
        sync.release(1);
    }

    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            //如果c==0 也就是没有重入锁了就直接完全释放。
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

    //入参是head节点
     private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
         //获取头部节点的status值如果小于0的话通过cas操作将值设置为0
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

当存在T2,T3线程的时候FairSync对象所维护的链表如图所示,

![](/assets/head-tail-Node.png)

总结 ：  
    1.锁状态。我们要知道锁是不是被别的线程占有了，这个就是 state 的作用，它为 0 的时候代表没有线程占有锁，  
    可以去争抢这个锁，用 CAS 将 state 设为 1，如果 CAS 成功，说明抢到了锁，这样其他线程就抢不到了，  
    如果锁重入的话，state进行+1 就可以，解锁就是减 1，直到 state 又变为 0，代表释放锁，  
    所以 lock\(\) 和 unlock\(\) 必须要配对啊。然后唤醒等待队列中的第一个线程，让其来占有锁。

```
2.线程的阻塞和解除阻塞。AQS 中采用了 LockSupport.park(thread) 来挂起线程，用 unpark 来唤醒线程。
3.阻塞队列。因为争抢锁的线程可能很多，但是只能有一个线程拿到锁，其他的线程都必须等待，
这个时候就需要一个 queue 来管理这些线程，AQS 用的是一个 FIFO 的队列，就是一个链表，
每个 node 都持有后继节点的引用。
```



