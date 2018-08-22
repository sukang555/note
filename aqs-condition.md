```java
    /**
     * Created by sukang on 2018/7/26.
     */

public class ThreadNotifyTest{

    /**
     * 定义一个volatile变量
     */
    private static  volatile int num = 0;

    private static Object lock = new Object();

    private static ReentrantLock reentrantLock = new ReentrantLock(true);
    private static Condition condition = reentrantLock.newCondition();


    @Test
    public void main3() throws Exception{
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                reentrantLock.lock();

                while (num != 1){
                    try {
                        System.out.println("不满足条件线程进入_WaitSet");
                        condition.await();

                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("满足条件，线程进入_EntryList");

            }
        },"BB");



        //线程A获取锁不满足条件进入等待队列
        thread.start();

        //为了保证线程A获取锁成功主线程先sleep
        TimeUnit.SECONDS.sleep(2);

        //线程A调用wait方法释放锁,主线程获取锁之后进行通知
        num = 1;
        reentrantLock.lock();
        //通知_WaitSet队列里的线程
        condition.signal();
        System.out.println("通知_WaitSet队列");

        //释放锁，然后让_WaitSet队列的线程再次获取锁
        reentrantLock.unlock();
    }




    /**
    *这种方式是通过Object的wait方法和notify方法来实现的
    *wait()、notify()是和synchronized配合使用的，因此如果使用了显示锁Lock，就不能用了。
    *所以显示锁要提供自己的等待/通知机制，Condition应运而生。
    */
    @Test
    public  void main2() throws  Exception{
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock){
                    while (num != 1){
                        try {
                            System.out.println("不满足条件线程进入_WaitSet");
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.println("满足条件，线程进入_EntryList");

                }
            }
        },"A");

        //线程A获取锁不满足条件进入等待队列
        thread.start();

        //为了保证线程A获取锁成功主线程先sleep
        TimeUnit.SECONDS.sleep(2);

        //线程A调用wait方法释放锁,主线程获取锁之后进行通知
        num = 1;
        synchronized (lock){
            //通知_WaitSet队列里的线程
            lock.notify();
            System.out.println("通知_WaitSet队列");
        }


    }







    /**
    *自旋实现的等待通知
    *最简单的实现方法就是将condition设为一个volatile的变量当A线程检测到条件不满足时就自旋
    *这种方式的问题在于自旋非常耗费CPU资源，当然如果在自旋的代码块里加入Thread.sleep(time)
    *将会减轻CPU资源的消耗，但是如果time设的太大，A线程就不能及时响应condition的变化，如果设的太小，
    *依然会造成CPU的消耗,因此我们可以改进通过notify来.


    *
    *
    *notifyAll：使所有获取过本锁对象的等待的线程统统退出wait的状态:
    *
    *notyfy :notify则文明得多他只是选择一个wait状态线程进行通知，并使它获得该对象上的锁，
    但不惊动其他同样在等待被该对象notify的线程
    *此时其它在等待队列的线程还是处于等待状态
    *
    /

    @Test
    public void main1() throws Exception{
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (num != 1){
                    System.out.println("不满足条件");
                }
                System.out.println("满足条件退出");
            }
        });

        thread.start();
        TimeUnit.SECONDS.sleep(2);

        num =1;

    }
}
```



![](/assets/ConditionObject.png)

//接下来我们看看Condition的实现源码

```java


    /** waitStatus value to indicate thread has cancelled */
    static final int CANCELLED =  1;
    /** waitStatus value to indicate successor's thread needs unparking */
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    static final int CONDITION = -2;
    /**
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
    static final int PROPAGATE = -3;
        
        
     public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            //调用await的线程，首先进入addConditionWaiter方法
            Node node = addConditionWaiter();
            //这里将线程获取的锁完全释放
            int savedState = fullyRelease(node);
            
            //判断当前线程是否还在wait队列中，如果存在就挂起线程。
            int interruptMode = 0;
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
        
    private Node addConditionWaiter() {
            Node t = lastWaiter;
            // 如果队列的尾部状态不是-2的话就将该Node从等待队列移除。
            if (t != null && t.waitStatus != Node.CONDITION) {
                unlinkCancelledWaiters();
                t = lastWaiter;
            }
            //将当前线程包装成Node对象添加到队列尾部，node状态为-2
            Node node = new Node(Thread.currentThread(), Node.CONDITION);
            if (t == null)
                firstWaiter = node;
            else
                t.nextWaiter = node;
            lastWaiter = node;
            return node;
        }
        
    //下边是释放当前线程占有的锁的源码
        
     final int fullyRelease(Node node) {
        boolean failed = true;
        try {
            //获取lock对象的锁状态
            int savedState = getState();
            if (release(savedState)) {
                failed = false;
                return savedState;
            } else {
                throw new IllegalMonitorStateException();
            }
        } finally {
            if (failed)
                node.waitStatus = Node.CANCELLED;
        }
    }
    
    //
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
        
           //获取锁的等待队列的头部对象
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    
    //获取当前锁的锁状态，这时c为0
     protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                //将锁属于的线程设置为null
                setExclusiveOwnerThread(null);
            }
            将锁状态这是为0
            setState(c);
            return free;
        }
        
        
        
        
        
//接下来看通知代码
     public final void signal() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            if (first != null)
                doSignal(first);
        }
        
      private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
        
     final boolean transferForSignal(Node node) {
        /*
         * If cannot change waitStatus, the node has been cancelled.
         */
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;

        /*
         * Splice onto queue and try to set waitStatus of predecessor to
         * indicate that thread is (probably) waiting. If cancelled or
         * attempt to set waitStatus fails, wake up to resync (in which
         * case the waitStatus can be transiently and harmlessly wrong).
         */
        Node p = enq(node);
        int ws = p.waitStatus;
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
        
        
```



