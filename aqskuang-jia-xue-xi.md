
---

AQS类图结构

![](/assets/2.png)

ReentrantLock类图结构

![](/assets/3.png)

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



