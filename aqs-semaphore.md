```java
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}

public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}
```



