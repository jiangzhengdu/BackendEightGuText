
回答：两种情况（基于JDK1.8）
===

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

1. 当HashMap中元素总个数达到阈值时就会扩容。注意是元素总个数，而不是数组占用个数。
数组扩容阈值,即：HashMap数组总容量 * 负载因子

int threshold
如果元素个数大于阈值，扩充数组。
if (++size > threshold)  
    resize();
2. 当向HashMap中添加元素时，即调用
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict)
方法时，如果通过hash值计算数组的索引，获取该索引位的首节点，且该首节点不为null时（即发生了Hash碰撞），会有对应三种处理情况：

 ① 新添加元素的key与首节点元素的key相同，即

// 如果首节点的key和要存入的key相同，那么直接覆盖value的值。
// p：hash值对应索引位置的首节点
if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
    e = p;

② 如果首节点是红黑树节点（TreeNode），将键值对添加到红黑树。

// 如果首节点是红黑树的，将键值对插添加到红黑树
else if (p instanceof HashMap.TreeNode)
    e = ((HashMap.TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);

③ 如果首节点是链表，将键值对添加到链表。添加之后会判断链表长度是否到达TREEIFY_THRESHOLD - 1这个阈值，“尝试”将链表转换成红黑树。

// p.next == null，到达链表末尾，添加新节点，如果长度足够，转换成树结构
if ((e = p.next) == null) {
    p.next = newNode(hash, key, value, null);  
    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
        treeifyBin(tab, hash);  
    break;
}
而关键在于这个treeifyBin()方法中，如果
// 把链表转换为红黑色树
final void treeifyBin(Node<K,V>[] tab, int hash) {
​    int n, index; Node<K,V> e;  // 如果当前数组容量太小（小于64），放弃转换，扩充数组。  
​    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY) {  
​        resize();  
​    } else if ((e = tab[index = (n - 1) & hash]) != null) {  
​        // 将链表转成红黑树...
​    }
}

* 每个节点要么是红色，要么是黑色，但根节点永远是黑色的；
* 每个红色节点的两个子节点一定都是黑色；
* 红色节点不能连续（也即是，红色节点的孩子和父亲都不能是红色）；
* 从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点；
* 所有的叶节点都是是黑色的（注意这里说叶子节点其实是上图中的 NIL 节点）；

HashMap在jdk1.8之后引入了红黑树的概念，表示若桶中链表元素超过8时，会自动转化成红黑树；
若桶中元素小于等于6时，树结构还原成链表形式。
而如果当前数组的容量太小（**小于64**），则放弃转换，扩充数组。扩充数组！

这样就有可能有一种情况出现，就是说我整个HashMap并没有达到负载因子那么多的元素个数，但是，我进行了一次扩容。所以单纯的以负载因子来看是否会发生扩容，是不全面的。

首先我们要理解HashMap为什么要进行扩容，扩容这个操作真正的意义在哪里？为什么要定义一个负载因子？设计他的初衷究竟是为了解决什么问题？
为什么要扩容呢，之所以要扩容，我理解是基于两个逻辑

1. HashMap中元素个数实在太多了，已经很频繁的发生hash冲突，我不得不进行扩容来减少这个冲突，来增强效率
2. HashMap中元素并不多，但元素都集中到那么一两个“坑位”上，数组占用的位置很少，但是在那一两个位置上的元素已经很多了，拖掉了执行的效率。
即核心是一个分布不均匀的问题，在这样的情况下，扩容的目的，就是通过重新计算hash“坑位”让原来聚集在一起的节点分散开来，是为了让分布更加均匀。
而treeifyBin()方法中，之所以判断当前数组容量是否太小，也是基于第二个逻辑上的考量，因为数组容量很小，但是已经存在过长的链表需要转换成树结构，
不如直接进行一次扩容，毕竟树的查询效率O(logn)也比不上一个正常HashMap的O(1)的效率
