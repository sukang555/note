```java
@Test
public void main2(){
    Map<String, Object> hashMap = new HashMap<>();

    hashMap.put("AA", "AA");
    hashMap.put("BB", "BB");  


    hashMap.get("");

}
```

![](/assets/HashMap.png)

1.table是一个Node类型的数组，Node的属性在下图；

2.size是已存放元素的数量

3.loadFactor是负载因子

4.entrySet是一个泛型为Entry的集合

![](/assets/Map.Node.png)

```java
//下边是HashMap中几个固定的静态常量值
static final float DEFAULT_LOAD_FACTOR = 0.75f;
static final int MAXIMUM_CAPACITY = 1 << 30;
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; //16
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
static final int MIN_TREEIFY_CAPACITY = 64;

//无参构造器中只是将负载因素的值赋值为，默认的负载因素，0.75f;
 public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
 }

 public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
 }


  //首先通过hash方法计算一个int类型的值，如果为null，key的hash值为0，
  static final int hash(Object key) {
        int h;   
        //这里可以这样理解 return hash^(hash >>> 16)

        //首先获取hash的低16位，再与本身做异或运算。  

        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }


  final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab;
        Node<K,V> p;
        int n, i;

        //如果table == null 或者 table的长度为0，则需要对table做初始化操作，n为初始化操作完成后的table的长度；
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;

        //根据(length - 1) & hash 求出这个key在当前数组的存放位置，也就是下标； 
        if ((p = tab[i = (n - 1) & hash]) == null)
           //如果当前位置为null,那么直接存放在当前位置
            tab[i] = newNode(hash, key, value, null);

        //如果计算出来的当前位置不为null，说明发生了碰撞;    
        else {
            Node<K,V> e; K k;

            //如果原来的key的hash值和新的hash值一样,并且==方法或者equals为true时e为旧的Node对象
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode){如果当前节点是红黑树那么就需要王红黑树添加节点；
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            }else {
                //如果p既不相等，也不属于红黑树，那么就是链表了，就需要往链表添加一个节点；
                for (int binCount = 0; ; ++binCount) {

                    //这里的p指的是发生碰撞索引下的旧的Node。

                    //从第一个开始，找到next为null的节点
                    if ((e = p.next) == null) {
                        //将新的Node放入到链表的尾部
                        p.next = newNode(hash, key, value, null);

                        //如果链表的长度 大于等于8的话，就变成红黑树存储；
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }

                    //这里链表的每一个都需要和当前key进行hash方法和equals方法进行判断，如果相同的话直接跳出循环;
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                        break;

                    如果不相同的话p更新为链表的下一个值，再进行下一次循环;    
                    p = e;
                }
            }


            //如果e不为null，说明e是已存在的key，这里只需将旧的value替换为新的value即可；
            if (e != null) { // existing mapping for key
                V oldValue = e.value;

                //onlyIfAbsent恒为true;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;

        这里如果添加完元素后，元素的数量 大于临界值的话就需要扩容;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
}



//扩容方法;

final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;

        //旧的容量
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //旧的临界值
        int oldThr = threshold;
        //新的容量和新的临界值
        int newCap, newThr = 0;

        //先看第一个if ，如果旧的容量大于0的话就需要在原来的基础上进行扩容
        if (oldCap > 0) {
            //如果旧的容量大于等于 1 << 30值，临界值为int的最大值，直接返回旧的数组
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }

            //如果旧的容量扩大两倍小于1 << 30 并且 旧的容量大于9的话新的临界值为原来临界值的两倍；

            //这里也是map的扩容策略，即每次都扩容为原来的2倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }

        //如果旧的容量等于0并且旧的临界值大于0，新的容量就是旧的临界值；
        else if (oldThr > 0){
            newCap = oldThr;
        }else {  //这里是调用第一次put时进入的判断，这里新的容量为默认值16；新的临界值为(int)(10*0.75) 也就是12;
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }

        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }

        //threshold指的就是当前的临界值；第一次调用完put时，值为12；
        threshold = newThr;

        //创建一个新的Node数组，大小为上边计算出来的新的容量值，第一次为16,以后每次扩容为原来的2倍；
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];

        //这里Map的table属性有了值;
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
 }



 //接下来我们再分析一下满足扩容条件时的扩容方法

 final Node<K,V>[] resize() {
        //旧的数组;
        Node<K,V>[] oldTab = table;
        //旧的数组的容量;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //旧的负载因素;
        int oldThr = threshold;
        //新的数组容量，新的负载因素;
        int newCap, newThr = 0;


        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }

            //这里新的容量为原来容量的2倍，新的负载因素也是原来负载因素的2倍；
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;

        //如果旧的数组不为null;
        if (oldTab != null) {


            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                //遍历旧的数组，如果当前位置不为null，取出当前位置的值，然后置为null
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    //如果当前位置的元素没有形成链表或者树，直接重新计算在新的数组的下标值，并赋值；
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    //如果当前位置的节点是红黑树的话执行split;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else {
                    //如果当前位置是链表的话 
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;

                            //这里求出最高位，如果是0的话就还是放到原来的位置；
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            //如果是1的话，就放到原来的索引+旧表长度的位置；
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
  }
```

我们总结一下这里边涉及到的算法分析；

1. hash = key.hashCode\(\)^\(key.hashCode\(\) &gt;&gt;&gt; 16\);
   index = hash & \(table.length -1\)

![](/assets/hash.png)

这里解释一下为什么我们选取数组的长度时选择2的正次幂，因为数组的长度 -1 相当于一个低位掩码，高位全部为0，低位全部为1.  
相当于保留hash的低位，计算后用来作为数组的下标。因此为了减少碰撞低位尽可能随机。

获取hash的值以后，拿高16位和原始的低16位做异或运算，结果是高16位保持不变，低16位也有了高16位的特征。

2.在进行除了首次以外的扩容方法时

![](/assets/hash2.png)

照道理，我在resize的时候，遍历所有的Node节点，获取他们的hash属性在于新的数组进行求与，重新计算数组下标。
但是如果是链表的话，说明它们的下标索引是相同的。因为扩容是原来的2倍 length - 1 就是在高位添加一个1；

0101 -> 5     00101 -> 5   10101 -> 16 + 5 = 21 

也就是说如果原来的hash高1位为0时，和新的数组大小求余以后取得的索引不变。高1位为1时，新的索引位置应该是原来的索引位置+旧的数组长度。

源码这样实现这个算法 if ((e.hash & oldCap) == 0) {}else{};






















