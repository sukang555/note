```java
Stack<E> extends Vector<E>

Stack 是集成自Vector;因此它具有Vector的全部属性;


@Test
public void main1(){
    Stack<Object> stack = new Stack<>();

    //向栈中压入元素
    stack.push("AA");
    stack.push("BB");
    stack.push("CC");
    //peek函数是从查看栈顶元素，但是不删除
    System.out.println(stack.peek());//CC
    //pop函数是移除并获取到栈顶元素
    //search是查找一个元素在Stack中的index
    System.out.println(stack.search("CC"));//1
}


//向栈内添加元素
public E push(E item) {
        addElement(item);

        return item;
}


//初始化完elementCount的值0;
 public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
}

private void ensureCapacityHelper(int minCapacity) {
        // 如果完成本次push所需的空间小于存放元素数组的长度就需要进行扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
}

private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
}



public synchronized E peek() {
        //获取元素的数量;
        int len = size();
        
        if (len == 0)
            throw new EmptyStackException();
        //栈顶的元素的索引也就是元素数量的最后一个值-1
        return elementAt(len - 1);
}

 public synchronized int search(Object o) {
      //获取元素的索引，例子中CC的索引为2，size是元素的数量，所以结果为1;也就说明Stack的元素栈顶是1，栈底的索引是元素的数量；
        int i = lastIndexOf(o);

        if (i >= 0) {
            return size() - i;
        }
        return -1;
}
 
 
 
public synchronized E pop() {
        E       obj;
        // 获取元素数量
        int     len = size();
        //获取栈顶元素
        obj = peek();
        
        移除指定索引的元素，一般就是数组的最后一个元素
        removeElementAt(len - 1);

        return obj;
}
 

  public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        
        
        // 这里计算j的值，我们pop的话应该是移除数组的最后一个j为0;
        
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
}
 
 
 
 
 
 
 
 
 
 
```



