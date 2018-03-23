我们知道，ArrayList的结构是数组，一般在程序中这样使用

```java
ArrayList<String> arrayList = new ArrayList<>();

arrayList.add("AA");
arrayList.add("BB");
arrayList.add("CC");


arrayList.get(0);
```

接下来我们先看一下ArrayList的属性再来一行一行来阅读源码。



![](/assets/qqq.png)

![](/assets/import2.png)

  该类 定义了两个空的Object数组，两个int类型变量，一个为null的临时Object数组；

```java
 public ArrayList() {
      //构造器中将一个空的object数组赋值给临时的elementData对象；
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
 }
 
 public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        }
}
 
 
 
 
```



