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




```



