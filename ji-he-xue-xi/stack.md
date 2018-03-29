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





```



	

