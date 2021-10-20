# **Collection接口**

集合的理解和好处：

>**数组**

* 长度必须指定，而且不能修改
* 保存的必须是同一个类型的元素
* 使用数组的时候进行增加、删除元素比较麻烦

>**集合**

* 可以动态保存任意多个对象
* 可以提供一系列的CRUd

> List,Set 是继承Collection接口，Collection接口继承的Iterable接口。

## **List**

ArrayList LinkedList  Vector 都实现了List接口

ArrayList子类的方法

* add()  可以添加任何Object子类
* remove() 可以删除指定元素  可以删除第n个元素、指定删除某个对象  
* contains() 如果存在 就返回true
* size()
* isEmpty()
* clear()
* addAll() 可以添加任何实现了Collection接口的对象
* containsAll() 判断任何实现了Collection接口的对象是否存在
* removeAll() 删除多个元素

## **Collection接口的遍历方式**

### **1. 使用迭代器 iterator**

所有实现了Collection接口的集合类都有一个iterator()的方法，用于返回一个实现了Iterator接口的对象，可以返回一个迭代器

Iterator仅用于便利集合，Iterator本省不存放对象

## **Set**

HashSet、TreeSet实现了Set接口
