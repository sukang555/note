TreeMap的底层是红黑树，而红黑树是一种近似平衡的二叉查找树，它能够确保任何一个节点的左右子树的高度差不会超过二者中较低那个的一陪。  
具体来说，红黑树是满足如下条件的二叉查找树（binary search tree）：

```
1.每个节点要么是红色，要么是黑色。

2.根节点必须是黑色

3.红色节点不能连续（也即是，红色节点的孩子和父亲都不能是红色）。

4.对于每个节点，从该点至null（树尾端）的任何路径，都含有相同个数的黑色节点。
```

红黑树的条件可能被破坏，需要通过调整使得查找树重新满足红黑树的条件。调整可以分为两类：一类是颜色调整，即改变某个节点的颜色；另一类是结构调整，集改变检索树的结构关系。结构调整过程包含两个基本操作：**左旋（Rotate Left），右旋（RotateRight）**

![](/assets/TreeMap.png)



![](/assets/TreeMap2.png)







```java
    @Test
    public void main3(){
        TreeMap<Object, Object> treeMap = new TreeMap<>();
        treeMap.put("1", "11");
        treeMap.put("2", "11");
        treeMap.put("3", "11");
        treeMap.put("4", "11");

        System.out.println(treeMap.get("2"));
     }
     
    public TreeMap() {
        comparator = null;
    }
    
    
     public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
```



