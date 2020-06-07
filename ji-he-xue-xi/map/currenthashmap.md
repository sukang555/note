我们知道HashMap不是线程安全的，因为两个线程同时操作一个hashMap的话，存在不同线程之间的值进行覆盖的现象，会导致数据丢失；下边我们看一下CurrentHashMap的源码；

![](/assets/CHashmap.png)

```java
static final int HASH_BITS = 0x7fffffff;
static final int MOVED     = -1;



public V put(K key, V value) {
        return putVal(key, value, false);
}
static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
}


final V putVal(K key, V value, boolean onlyIfAbsent) {
        //如果key为null或者value为null,抛出异常
        if (key == null || value == null) throw new NullPointerException();

        // 得到hash的值
        int hash = spread(key.hashCode());

        int binCount = 0; 

        //进入一个循环
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f;
            int n, i, fh;
            //如果tab为null或者length长度为0，就初始化tab对象
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();

            //如果计算出的索引位置为null的话进行cas赋值当前下标。如果失败的话说明有其它线程
            在这一时刻给这个下标赋值，因此不会break，就会进入下个循环；
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            //如果当前下标的Node的hash值为-1 ，没看懂这里；
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);

            else {
              //f为当前下标的Node对象
                V oldVal = null;
                //这里获取当前下表位置的头部对象锁
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {//fn指的是头部节点的hash值
                            //而且这个if里边做的链表的逻辑

                            //binCount 指的是链表的长度；
                            binCount = 1;
                            //遍历这个链表，如果链表中已存在相同的key，只用将值覆盖为最新的。
                            //否则，直到找到链表的尾部，将新的Node放到尾部
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        //如果f属于树的话，就往树里添加对象
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                //如果走的链表的逻辑binCount的值为链表的长度，
                //如果走的是红黑树的逻辑binCount的值为固定值2，
                if (binCount != 0) {
                    //如果binCount的值大于等于8，就变换成红黑树
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
}
```



