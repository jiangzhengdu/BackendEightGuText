MongoDB和Mysql的区别
===

MySQL是关系型数据库。

优势
---

在不同的引擎上有不同的存储方式。

查询语句是使用传统的sql语句，拥有较为成熟的体系，成熟度很高。

开源数据库的份额在不断增加，mysql的份额页在持续增长。

缺点
---

在海量数据处理的时候效率会显著变慢。

Mongodb是非关系型数据库(nosql),属于文档型数据库。文档是mongoDB中数据的基本单元，类似关系数据库的行，多个键值对有序地放置在一起便是文档，
语法有点类似javascript面向对象的查询语言，它是一个面向集合的，模式自由的文档型数据库。

存储方式：虚拟内存+持久化。

查询语句：是独特的Mongodb的查询方式。

适合场景：事件的记录，内容管理或者博客平台等等。

架构特点：可以通过副本集，以及分片来实现高可用。

数据处理：数据是存储在硬盘上的，只不过需要经常读取的数据会被加载到内存中，将数据存储在物理内存中，从而达到高速读写。

成熟度与广泛度：新兴数据库，成熟度较低，Nosql数据库中最为接近关系型数据库，比较完善的DB之一，适用人群不断在增长。

优点
---

快速！在适量级的内存的Mongodb的性能是非常迅速的，它将热数据存储在物理内存中，使得热数据的读写变得十分快。高扩展性，存储的数据格式是json格式
缺点
---

不支持事务，而且开发文档不是很完全，完善。

Mysql和Mongodb主要应用场景
---

1.如果需要将mongodb作为后端db来代替mysql使用，即这里mysql与mongodb 属于平行级别，那么，这样的使用可能有以下几种情况的考量：
(1)mongodb所负责部分以文档形式存储，能够有较好的代码亲和性，json格式的直接写入方便。(如日志之类)
(2)从datamodels设计阶段就将原子性考虑于其中，无需事务之类的辅助。开发用如nodejs之类的语言来进行开发，对开发比较方便。
(3)mongodb本身的failover机制，无需使用如MHA之类的方式实现。

2.将mongodb作为类似redis ，memcache来做缓存db，为mysql提供服务，或是后端日志收集分析。考虑到mongodb属于nosql型数据库，
sql语句与数据结构不如mysql那么亲和，也会有很多时候将mongodb做为辅助mysql而使用的类redis memcache 之类的缓存db来使用。
亦或是仅作日志收集分析。
