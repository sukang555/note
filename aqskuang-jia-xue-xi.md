





![](/assets/1.png)











AbstractQueuedSynchronizer 这个类如上图所示，tail代表尾部节点，head代表头部节点，state属性表示有无线程占有。

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

