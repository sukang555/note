

```java

    //map的在put的时候存在一个将数组
//这是HashMap的红黑树源码。
final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index;
        Node<K,V> e;
        
        //这里tab的长度小于64的话只进行扩容操作，没有做红黑树转换
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
}

```

