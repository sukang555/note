![](/assets/Vector.png)

```java
    //一般在程序中我们这样使用vector
    List<String> vector = new Vector<>();
    vector.add("SS");
    vector.add("DD");
    vector.add("FF");
    vector.get(2);


    public Vector() {
        this(10);
    }


    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }


    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }


    //从vector的构造器中可以看到elementData初始化为一个容量为10的object数组，capacityIncrement为0；


    //vector在add的方法上添加了synchronized关键字，来保证线程安全；
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }


    //如果数组长度不够存放新添加的元素，就需要进行扩容操作，这点和ArrayList一样；
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {

        //获取旧的数组长度
        int oldCapacity = elementData.length;
        //扩容策略为。增长长度大于0的话就为旧的长度添加增长长度。否则扩大为原来的2倍；
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```



