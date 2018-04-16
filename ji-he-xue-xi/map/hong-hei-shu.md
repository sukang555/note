






```java
//map的在put的时候存在一个将数组
//这是HashMap的红黑树源码。
final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index;
        Node<K,V> e;

        //这里tab的长度小于64的话只进行扩容操作，没有做红黑树转换
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();

        // 这里e为链表的头部，   
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;


            do {
                //根据链表的头部节点创建一个treeNode节点
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);

            //上边的do{}while() 语句将Node节点链表转换成TreeNode节点的链表

            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
}


TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        return new TreeNode<>(p.hash, p.key, p.value, next);
}

 TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
}


//这里将treeNode的链表转换为红黑树;
final void treeify(Node<K,V>[] tab) {
            //红黑树的根部节点
            TreeNode<K,V> root = null;

            //这里的this指的是链表的头部；x指的是root节点
            for (TreeNode<K,V> x = this, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;

                //初始化root节点，颜色为黑色
                if (root == null) {
                    x.parent = null;
                    x.red = false;
                    root = x;
                }

                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        K pk = p.key;
                        //大于根节点的hash,放右边，小于放左边
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);

                        TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            root = balanceInsertion(root, x);
                            break;
                        }
                    }
                }
            }
            moveRootToFront(tab, root);
}
```

![](/assets/TreeNode.png)

