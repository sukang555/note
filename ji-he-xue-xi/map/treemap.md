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
        //第一次put的时候根节点root的值为null
        Entry<K,V> t = root;
        if (t == null) {
            compare(key, key); // type (and possibly null) check
            //这个时候就初始化root节点
            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths

        //这个地方获取默认的comparator对象，因为在构造器中我们可以给comparator初始化自己的比较策略：

        //未初始化的话comparator对象为null
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

         //comparator为null的话需要保证key不为null;
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;

            // t为当前的节点，第一次循环为root根节点,一直循环到当前节点的最后子节点为null的时候，
            // 最后parent的值为子节点为null的节点;
            do {
                parent = t;

                //将新添加的key与当前遍历到的节点的key进行比较
                cmp = k.compareTo(t.key);
                //小于走左边，大于走右边，等于覆盖当前节点的值；
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }

        //最后再将新添加的key包装为Entry放到最后位置;这个时候新添加过程已经完成
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        //这个方法是新添加完成以后要验证是否符合红黑树的特征，然后在进行左旋右旋或者改变颜色操作
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
```

下边我们通过例子来分析fixAfterInsertion\(\)方法

![](/assets/Tree4.png)

                 图 1.1





![](/assets/TreeMap1.2.png)

                       图 1.2

```java
    //我们看到上图中，根节点是黑色，红色也不连续，当我们新加入15元素的时候
    //x这里指的是新添加的元素
     private void fixAfterInsertion(Entry<K,V> x) {
        //新添加的元素的颜色为红色
        x.color = RED;
        //由于15的父亲元素10为红色，因此会进入这个循环
        while (x != null && x != root && x.parent.color == RED) {

            //如果新插入节点x的父节点是祖父节点的左孩子
            if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
                //获取该节点的祖父的右子树
                Entry<K,V> y = rightOf(parentOf(parentOf(x)));
                //如果右子树为红色
                if (colorOf(y) == RED) {
                  //则将该节点的祖父节点的左右子书全部置为黑色，祖父节点的颜色置为红色
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    if (x == rightOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateLeft(x);
                    }
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateRight(parentOf(parentOf(x)));
                }
            } else {
            
             //如果新插入节点x的父节点是祖父节点的右孩子
                Entry<K,V> y = leftOf(parentOf(parentOf(x)));
                //如果该节点的祖父节点的左子节点的颜色为红色 图1.1
                if (colorOf(y) == RED) {
                    //将该节点的父类节点全部置为黑色，并将祖父节点置为红色图1.2
                    setColor(parentOf(x), BLACK);
                    setColor(y, BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    x = parentOf(parentOf(x));
                } else {
                    if (x == leftOf(parentOf(x))) {
                        x = parentOf(x);
                        rotateRight(x);
                    }
                    setColor(parentOf(x), BLACK);
                    setColor(parentOf(parentOf(x)), RED);
                    rotateLeft(parentOf(parentOf(x)));
                }
            }
        }
        //根节点一直为黑色
        root.color = BLACK;
    }
```



