HashMap
===

Hashmap的遍历
---

```java
HashMap<Integer,Integer> map = new HashMap<Integer, Integer>();

    for(Integer i:map.keySet()){
        System.out.println("key="+i+" value="+map.get(i));
    }

    Iterator iterator1=map.entrySet().iterator();
  while(iterator1.hasNext()){
      int key=(int)iterator1.next();
      int value=(int)map.get(key);
  }
```

```java
    Map<String, String> map = ...
    for (Map.Entry<String, String> entry : map.entrySet()) {
        System.out.println(entry.getKey() + "/" + entry.getValue());
    }
```

jdk1.7的HashMap底层实现原理
---

HashMap map = new HashMap():

在实例化以后，底层创建了长度是16的一维数组Entry[] table。
…可能已经执行过多次put…
map.put(key1,value1):
首先，调用key1所在类的hashCode()计算key1哈希值，此哈希值经过某种算法计算以后，得到在Entry数组中的存放位置。
如果此位置上的数据为空，此时的key1-value1添加成功。 ----情况1
如果此位置上的数据不为空，(意味着此位置上存在一个或多个数据(以链表形式存在)),比较key1和已经存在的一个或多个数据的哈希值：
    5.1： 如果key1的哈希值与已经存在的数据的哈希值都不相同，此时key1-value1添加成功。----情况2
    5.2 ： 如果key1的哈希值和已经存在的某一个数据(key2-value2)的哈希值相同，继续比较：调用key1所在类的equals(key2)方法，比较：
    5.2.1 ：如果equals()返回false:此时key1-value1添加成功。----情况3
    5.2.2 ：如果equals()返回true:使用value1替换value2。

补充：关于情况2和情况3：此时key1-value1和原来的数据以链表的方式存储。
在不断的添加过程中，会涉及到扩容问题，当超出临界值(且要存放的位置非空)时，扩容。默认的扩容方式：扩容为原来容量的2倍，并将原有的数据复制过来

jdk1.8的HashMap底层实现原理
---

jdk8 相较于jdk7在底层实现方面的不同

new HashMap():底层没有创建一个长度为16的数组
    jdk 8底层的数组是：Node[],而非Entry[]
    首次调用put()方法时，底层创建长度为16的数组
    jdk7底层结构只有：数组+链表。jdk8中底层结构：数组+链表+红黑树。
    4.1 形成链表时，七上八下（jdk7:新的元素指向旧的元素。jdk8：旧的元素指向新的元素）
    4.2 当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8 且当前数组的长度 > 64时，此时此索引位置上的所数据改为使用红黑树存储。
    DEFAULT_INITIAL_CAPACITY : HashMap的默认容量，16
    DEFAULT_LOAD_FACTOR：HashMap的默认加载因子：0.75
    threshold：扩容的临界值，= 容量 cheng 填充因子16 * 0.75 => 12
    TREEIFY_THRESHOLD：Bucket中链表长度大于该默认值，转化为红黑树:8
    MIN_TREEIFY_CAPACITY：桶中的Node被树化时最小的hash表容量:64

HashMap线程不安全的体现
---

jdk1.7中的HashMap

在jdk1.8中对HashMap做了很多优化，这里先分析在jdk1.7中的问题，相信大家都知道在jdk1.7多线程环境下HashMap容易出现死循环，这里我们先用代码来模拟出现死循环的情况：
从堆栈信息中可以看到出现死循环的位置，通过该信息可明确知道死循环发生在HashMap的扩容函数中，根源在transfer函数中，jdk1.7中HashMap的transfer函数如下：
在对table进行扩容到newTable后，需要将原来数据转移到newTable中，注意10-12行代码，这里可以看出在转移元素的过程中，使用的是头插法，也就是链表的顺序会翻转，这里也是形成死循环的关键点。下面进行详细分析

前提条件：
这里假设

1. hash算法为简单的用key mod链表的大小
2. 最开始hash表size=2，key=3,7,5，则都在table[1]中。
3. 然后进行resize，使size变成4。

在jdk1.7中，在多线程环境下，扩容时会造成环形链或数据丢失

jdk 1.8:
        if ((p = tab[i = (n - 1) & hash]) == null) // 如果没有hash碰撞则直接插入元素
这是jdk1.8中HashMap中put操作的主函数， 注意第6行代码，如果没有hash碰撞则会直接插入元素。如果线程A和线程B同时进行put操作，刚好这两条不同的数据hash值一样，并且该位置数据为null，所以这线程A、B都会进入第6行代码中。假设一种情况，线程A进入后还未进行数据插入时挂起，而线程B正常执行，从而正常插入数据，然后线程A获取CPU时间片，此时线程A不用再进行hash判断了，问题出现：线程A会把线程B插入的数据给覆盖，发生线程不安全。
在jdk1.8中，在多线程环境下，会发生数据覆盖的情况。

为什么HashTable慢
-----

Hashtable之所以效率低下主要是因为其实现使用了synchronized关键字对put等操作进行加锁，而synchronized关键字加锁是对整个对象进行加锁，也就是说在进行put等修改Hash表的操作时，锁住了整个Hash表，从而使得其表现的效率低下。
