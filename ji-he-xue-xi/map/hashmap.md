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
            
        //当前数组的长度   
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
            
        //    
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
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
        
        //如果旧的临界值大于0，新的容量就是旧的临界值；
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
        
        //这里Map的table属性有了值；
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


```






















