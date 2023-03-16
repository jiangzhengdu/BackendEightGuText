Java集合的快速失败机制 “fail-fast”？
===

简单来说就是系统运行中，如果有错误发生，那么系统立即结束，而不是继续冒不确定的风险继续执行，这种设计就是"快速失败"
在使用迭代器遍历集合时，如果迭代器创建之后，通过 除了迭代器提供的修改方法之外 的其他方式对集合进行了结构性修改（添加、删除元素等），那么迭代器应该抛出一个ConcurrentModificationException异常，表示在此次遍历中集合发生了"并发修改"，应该提前终止迭代过程。因为在迭代器遍历集合的过程中，如果有别的行为改变了集合本身的结构，那么迭代器之后的行为可能就是不符合预期的，可能会出现错误的结果，所以提前检测并抛出异常是一个更好的做法。

我们知道Java中Collection接口下的很多集合都是线程不安全的, 比如 java.util.ArrayList不是线程安全的, 因此如果在使用迭代器的过程中有其他线程修改了list，那么将抛出ConcurrentModificationException，这就是所谓fail-fast策略。

这一策略在源码中的实现是通过 **modCount**域，modCount 顾名思义就是修改次数，对ArrayList 内容的修改都将增加这个值，那么在迭代器初始化过程中会将这个值赋给迭代器的 expectedModCount。在迭代过程中，判断 modCount 跟 expectedModCount 是否相等，如果不相等就表示已经有其他线程修改了 list
注意到 modCount 声明为 volatile，保证线程之间修改的可见性

modCount和expectedModCount
---

modCount和expectedModCount是用于表示修改次数的，其中modCount表示集合的修改次数，这其中包括了调用集合本身的add, remove, clear方法等修改方法时进行的修改和调用集合迭代器的修改方法进行的修改。而expectedModCount则是表示迭代器对集合进行修改的次数。
设置expectedModCount的目的就是要保证在使用迭代器期间，list对象只能有这一个迭代器对list进行修改
如果创建多个迭代器对一个集合对象进行修改的话，那么就会有一个modCount和多个expectedModCount，且modCount的值之间也会不一样，这就导致了moCount和expectedModCount的值不一致，从而产生异常
也就是在对集合进行数据的增删的时候都会执行modcount++, 那么如果一个线程还在使用迭代器遍历这个list的时候就会发现异常, 发生fail-fast(快速失败)

fail-fast(快速失败)和fail-safe(安全失败)比较
---

Iterator的快速失败是基于对底层集合做拷贝是浅拷贝，因此，它受源集合上修改的影响。j
ava.util包下面的所有的集合类都是快速失败的

而java.util.concurrent包下面的所有的类都是使用锁实现安全失败的。

快速失败的迭代器会抛出ConcurrentModificationException异常，而安全失败的迭代器永远不会抛出这样的异常。

fail-fast解决什么问题
---

fail-fast机制，是一种错误检测机制。
它只能被用来检测错误，因为JDK并不保证fail-fast机制一定会发生。只是在多线程环境下告诉客户端发生了多线程安全问题.
所以若在多线程环境下使用fail-fast机制的集合，建议使用“java.util.concurrent包下的类”去取代“java.util包下的类”。
如何解决fail-fast事件

ArrayList对应的CopyOnWriteArrayList进行说明。我们先看看CopyOnWriteArrayList的源码：
CopyOnWriteArrayList是自己实现Iterator, 并且CopyOnWriteArrayList的Iterator实现类中，没有所谓的checkForComodification()，更不会抛出ConcurrentModificationException异常
CopyOnWriteArrayList在进行新建COWIterator时，将集合中的元素保存到一个新的拷贝数组中。这样，当原始集合的数据改变，拷贝数据中的值也不会变化。

怎么确保一个集合不能被修改？
---

可以使用 Collections. unmodififiableCollection(Collection c) 方法来创建一个只读集合，这样改变集合的任何操作都会抛出 Java. lang. UnsupportedOperationException 异常。
示例代码如下：

如何边遍历边移除 Collection 中的元素
---

边遍历边修改 Collection 的唯一正确方式是使用 Iterator.remove() 方法，如下：

``` Java
Iterator<Integer> it = list.iterator();
while(it.hasNext()){
   *// do something*
   it.remove();
}
```

一种最常见的错误代码

``` java
for(Integer i : list){
   list.remove(i)
}
```

 运行以上错误代码会报 ConcurrentModificationException 异常。这是因为当使用 foreach(for(Integer i : list)) 语句时，会自动生成一个iterator 来遍历该 list，但同时该 list 正在被 Iterator.remove() 修改。Java 一般不允许一个线程在遍历 Collection 时另一个线程修改它。

使用foreach遍历List，但不能对某一个元素进行操作（这种方法在遍历数组和Map集合的时候同样适用）

Iterator 和 ListIterator 有什么区别？
---

Iterator 可以遍历 Set 和 List 集合，而 ListIterator 只能遍历 List。
Iterator 只能单向遍历，而 ListIterator 可以双向遍历（向前/后遍历）。
ListIterator 实现 Iterator 接口，然后添加了一些额外的功能，比如添加一个元素、替换一个元素、获取前面或后面元素的索引位置。

Java中的遍历（遍历集合或数组的几种方式）
---

1. 使用下标遍历
使用循环遍历数组下标范围，在循环体中用下标访问数组元素：

``` java
for (int i = 0; i < array.length; i++) {
    System.out.println(array[i]);
}
```

2.增强for循环

使用增强for循环语法结构来对数组进行遍历：

``` java
for (int value : array) {
    System.out.println(value);
}
```

增强for循环 其实只是一种语法糖，使用 增强for循环 在遍历数组时，在编译过程会将其转化为 "使用下标遍历" 的方式，在字节码层面其实等价于第一种方式，效率上也没有太大差别。

JAVA中的迭代器
---

在 面向对象编程里，迭代器模式是一种设计模式，是一种最简单也最常见的设计模式。迭代器模式提供了一种方法顺序访问一个集合对象中的各个元素，而又不暴露其内部的表示。Java中也提供了对迭代器模式的支持，主要是针对Java的各种集合类进行遍历
