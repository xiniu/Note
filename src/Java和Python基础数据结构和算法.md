# Java

## 数组

- 和C++不同，leetcode上如果需要传递数组类型参数，用的还是原生数组；可能是因为Java的原生数组能获取到length
- 声明和定义`int [] arr = {1,2,3,4}`或者`int [] arr = {1,2,3}`
- 遍历 `for (int i = 0; i < arr.length; i++){System.out.println(arr[i]);}`或者`for(int e:arr){System.out.println(e);}`
- 通过使用java.util.Arrays提供的一些方法方便的操作数组；填充值`Arrays.fill(arr,0)`;排序`Arrays.sort(arr)`

## 常见的接口

- Java不同于C++的地方，对于常见数据结构的使用，有接口的概念；实现接口的实现类可能各部相同，但是都要实现该接口规定的方法
- List接口常见的方法有：
  - 获取大小 `int size()`和判空`bool ieEmpty()`
  - 是否包含某一元素`bool contains(Object o)`和包含传入集合的所有元素`bool containsAll(collection<?>c)`
  - 返回迭代对象`Iterator<E> iterator()`
  - 添加元素`bool add(object o)`、删除元素(如果包含则删除最先找到的一个)`bool remove(object o)`和添加传入集合所有元素`bool addAll(collection<?>c)`
  - 获取普通数组`object[] toArray()`(与之对应的`Arrays.asList(object[])`)
-   
