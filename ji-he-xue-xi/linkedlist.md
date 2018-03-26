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
 
 
 
 
 
```



