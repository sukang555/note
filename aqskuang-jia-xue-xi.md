
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

            //如果FairSync的state不等于0的话再判断一下当前线程是否已经获取过锁了，如果已经或去过那么就state+1，这也就是重入锁。返回true；

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
            //获取T2线程的state的值，此时获取的state的值大于0 我们之前分析过T1，T1线程获取完线程以后会将state的值通过cas方式加1操作，
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

        //我们又回到这个方法，T2前一个判断返回的是false;那么程序会执行addWaiter()方法,并将返回结果当作参数继续执行acquireQueued()方法；
        //现在我们看Node的属性

private Node addWaiter(Node mode) {

        //此时new了node，mode为null 这里我们看一下这个构造器
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
        return node;
}

private Node enq(final Node node) {

        //此时T2进入这里，首先进入一个死循环这种方式编译出来的指令要比while少。
        for (;;) {
        //这个里 tail为null，传进来的node就是刚刚new出来的节点对象。
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
}




```



