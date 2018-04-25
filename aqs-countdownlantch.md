//我们先看一个例子

```java
    public static void  main(String[] args){
        CountDownLatch latch = new CountDownLatch(2);

        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (Exception e) {
                e.printStackTrace();
            }
            latch.countDown();
        },"T1").start();


        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 休息 10 秒后(模拟线程工作了 10 秒)，调用 countDown()
            latch.countDown();

        }, "T2").start();



        new Thread(() -> {
            try {
                // 阻塞，等待 state 减为 0
                latch.await();
                System.out.println("线程 t3 从 await 中返回了");
            } catch (InterruptedException e) {
                System.out.println("线程 t3 await 被中断");
                Thread.currentThread().interrupt();
                e.printStackTrace();
            }

        }, "T3").start();


        new Thread(() -> {
            try {
                // 阻塞，等待 state 减为 0
                latch.await();
                System.out.println("线程 t4 从 await 中返回了");
            } catch (InterruptedException e) {
                System.out.println("线程 t4 await 被中断");
                Thread.currentThread().interrupt();
                e.printStackTrace();
            }

        }, "T4").start();

    }
    //上边例子中有4个线程，T1和T2调用countDown方法将state的值进行减一操作，
    //T3,T4线程调用await方法，线程进入阻塞等待state的值减到0为止，然后T3,T4唤醒继续走.


    public void countDown() {
        sync.releaseShared(1);
    }

    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
        //只有减一操作后的state的值为0，才会进入这个方法
            doReleaseShared();
            return true;
        }
        return false;
    }

    //这个方法是采用子旋的方式将state的值减一操作
    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
            for (;;) {
            //获取state的值
                int c = getState();
                //如果state直接等于0,说明不能进行减一操作直接返回.
                if (c == 0)
                    return false;
                int nextc = c-1;
                //进行cas将减一后的值重新赋值，如果失败说明有另外线程已经赋值，
                //所以进入下一个循环
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
    }

     private void doReleaseShared() {
     
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```


