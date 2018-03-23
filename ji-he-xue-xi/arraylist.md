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
          //如果大于0的话，elementData初始化一个Object数组，大小为传入的值的大小。
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
        //如果等于0的话，elementData就为一个空的数组；
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        }
}


//数组添加第一个对象
  public boolean add(E e) {
      //这里是扩容方法
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        
        //这里可以分解为两步
        //elementData[size] = e;
        // ++size
        elementData[size++] = e;
        return true;
  }
   
   
  private void ensureCapacityInternal(int minCapacity) {
        //如果elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA 说明是调用的无参构造器。
        //数组没满之前minCapacity都是默认值10，数组满的话minCapacity就是当前size+1的值；
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
}

 private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
}
 
 
```



