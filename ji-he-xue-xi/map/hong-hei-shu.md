

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


TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        return new TreeNode<>(p.hash, p.key, p.value, next);
}

 TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
}


```

