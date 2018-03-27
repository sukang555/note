![](/assets/LinkList.png)

![](/assets/LinkList-Node.png)

上边的是LinkedList 的类结构图。LinkedList一般我们在程序中这样使用

```java
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("AA");
linkedList.add("BB");

linkedList.get(0);
```

```java
//先看一下LinkedList的无参构造器
 public LinkedList() {
 //无参构造器里无操作
 }

 public boolean add(E e) {
        //调用add方法，进入linkLast方法
        linkLast(e);
        return true;
 }

 void linkLast(E e) {
        //将last的值赋值给l，这里第一次调用add的话l的值为null；
        final Node<E> l = last;
        //将add的元素封装为一个Node，这里新的Node的item为当前元素，下一个为null，上一个为LinkedList的last元素，当然初始为null;
        final Node<E> newNode = new Node<>(l, e, null);
        //更新LinkedList对象的last的值为新元素封装成的Node;
        last = newNode;

        //如果l为null，说明是第一次添加元素，那么LinkedList的first属性就为刚才新的Node;
        if (l == null)
            first = newNode;
       //如果不为null，那么说明当前的last的下一个就是新元素的Node;
        else
            l.next = newNode;
        size++;
        modCount++;
}


Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
}



public E get(int index) {
        //先检查指定索引
        checkElementIndex(index);
        return node(index).item;
}

Node<E> node(int index) {
        //如果需要查找的索引位置位于前半部分则从头部开始往后找，如果位于后半部分则从尾部往前找

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
}


//移除操作
 public E remove(int index) {
        checkElementIndex(index);

        //先获取指定索引的Node节点
        return unlink(node(index));
}

E unlink(Node<E> x) {
        // 
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        //如果前一个为null说明是移除的是头部节点；
        if (prev == null) {
            //更新集合的first节点为移除的下一个
            first = next;
        } else {
         //不为头部节点，被移除节点的前一个的下一个更新为当前节点的下一个；被移除节点的前一个置为null
            prev.next = next;
            x.prev = null;
        }

        //如果next为null，说明移除的是尾部节点，然后更新集合的最后节点

        if (next == null) {
            last = prev;
        } else {
        //不为尾部节点，移除节点的下一个的上一个更新为移除节点的上一个
            next.prev = prev;
            x.next = null;
        }
        //size减一操作，操作次数累加
        x.item = null;
        size--;
        modCount++;
        return element;
}
```

1.LinkedList 内部维护了一个双向链表，head和last分别为链表的头部和尾部；  
2.LinkedList 查找指定索引的元素效率低，是因为需要从头部节点开始往后遍历；  
3.LinkedList 查找元素分别从头部往后遍历到中间，或者从尾部往前遍历到中间；  
4.LinkedList 移除元素要比ArrayList要快因为只用更新被移除节点的前一个节点的下一个引用，以及下一个节点的上一个引用；

