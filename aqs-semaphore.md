```java
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}

public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

//Semaphore也是两个构造器，分别返回公平锁对象或者是非公平锁对象
```



