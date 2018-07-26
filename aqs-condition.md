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
    *依然会造成CPU的消耗,因此我们可以改进通过notyfy来.
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



