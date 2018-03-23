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

        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
       //minCapacity可以理解为需要完成这次add所需要的最小容量。
        ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        //如果 elementData.length < minCapacity 
        //当前数组的长度 小于 完成add所需要的最小容量 的话就需要进行扩容操作。也就是说数组没有足够的空间来存放数据了。
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
}

 private void grow(int minCapacity) {
        // overflow-conscious code
        //获取旧的数组的长度
        int oldCapacity = elementData.length;

        //新的长度 = old + (int)(old/2)

        int newCapacity = oldCapacity + (oldCapacity >> 1);

        //如果计算出来的新的容量小于所需的最小容量的话新的就被赋值为所需的最小容量。
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;

        // 如果计算出来的新的容量大于int的最大值-8的话，走huge方法；所需最小容量大于int最大值-8就是int最大值，否则就是int最大值-8   
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);

        //重新创建一个新容量的数组将旧的数组内容copy进来并赋值给当前list数组。
        elementData = Arrays.copyOf(elementData, newCapacity);
}






private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
}
```

1.初始化时 ： 数组为{} 因此数组长度为0；

2.第一次调用add时，我需要的数组容量为1，此时数组的长度为0 因此需要扩容操作，第一次扩容出来的长度为默认值10;

![](file:///C:\Users\hand\AppData\Roaming\Tencent\Users\1677931994\TIM\WinTemp\RichOle\C]9}D[44JW184%D%29MDB~BHW.png)

3.当第11次调用add方法时 需要的容量为11 此时就会进入扩容操作，新的容量为 10 + (int)(10/2) = 15;

