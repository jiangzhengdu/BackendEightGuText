定位慢SQL
===

一、整体思路
---

1.根据慢日志定位慢查询sql
---

使用 show varables like %query% , 查询变量
show status like '%slow_queries%'  , 慢查询的数量
set global slow_query_log = on ; 打开慢查询
set global long_query_time = 1 ；设置慢查询的时间阈值为：1s
假设下面这条查询产生慢查询（2000000条数据）
select count(id) from person_info_large order by name desc；
再次执行show status like '%slow_queries%'
产生了一条慢日志
通过终端去查看该日志

2.使用explain等工具分析sql
---

使用explain分析

关键字一般放在select的前面，用于描述maysql如何执行操作，已经mysql成功返回需要执行的函数，他可以帮我们分析select语句。帮助我们查找到查询小效率低下的原因，让查询优化器更好的工作

可以获取到更为详细的信息：

出现以下两项表示mysql无法使用索引，应尽可能优化

using temportory 表示在mysql中对查询结果排序的时候，使用临时表，常见于使用group by order by
using filesort   表示mysql会使用一个外部索引排序 而不是从表里的按照索引次序读到相关内容，可能在内存和磁盘上进行排序，mysq无法使用这个排序的缩影完成的排序叫做 using filesort

1. id ：select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序 **id越大**的table读取越在前面吗
2. select_type ：查询类型 或者是 其他操作类型
    * SIMPLE 简单查询，不包括子查询和union查询
    * PRIMARY 当存在子查询时，最外面的查询被标记为主查询
    * SUBQUERY 子查询
    * UNION 当一个查询在UNION关键字之后就会出现UNION
    * UNION RESULT 连接几个表查询后的结果
    * DERIVED 衍生表，而衍生表的命名就是<select_type + id>
3. table ：正在访问哪个表 读取顺序是自上向下的
4. partitions ：匹配的分区 该列显示的为分区表命中的分区情况。非分区表该字段为空（null）。
5. type ：访问的类型 优化从**左到右递减**
**NULL > system > const > eq_ref > ref > ref_or_null > index_merge > range > index > ALL**
    * NULL MySQL能够在优化阶段分解查询语句，在执行阶段用不着再访问**表或索引** 比如当你要查询最大值或者最小值时，MySQL会直接到你的索引得分叶子节点上**直接拿，所以不用访问表或者索引**。但是！你要记住，NULL的前提是你已经建立了索引。
    * SYSTEM 表只有一行记录（等于系统表），这是const类型的特列，平时不大会出现，可以忽略
    * const 表示通过索引一次就找到了，const用于比较primary key或uique索引，因为只匹配一行数据，所以很快，如主键置于where列表中，MySQL就能将该查询转换为一个常量。简单来说，const是直接按主键或唯一键读取
    * eq_ref 用于联表查询的情况，按联表的主键或唯一键联合查询。多表join时，对于来自前面表的每一行，在当前表中只能找到一行。这可能是除了system和const之外最好的类型。当主键或唯一非NULL索引的所有字段都被用作join联接时会使用此类型。
    * ref 可以用于单表扫描或者连接。如果是连接的话，驱动表的一条记录能够在被驱动表中通过非唯一（主键）属性所在索引中匹配多行数据，或者是在单表查询的时候通过非唯一（主键）属性所在索引中查到一行数据
    * ref_or_null 类似ref，但是可以搜索值为NULL的行
    * index_merge 表示查询使用了两个以上的索引，最后取交集或者并集，常见and ，or的条件使用了不同的索引，官方排序这个在ref_or_null之后，但是实际上由于要读取多个索引，性能可能大部分时间都不如range。
    * range 索引范围查询，常见于使用 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN()或者like等运算符的查询中。
    * index index只遍历索引树，通常比All快。因为，索引文件通常比数据文件小，也就是虽然all和index都是读全表，但index是从索引中读取的，而all是从硬盘读的。
    * ALL 如果一个查询的type是All,并且表的数据量很大，那么请解决它！！
6. possible_keys ：显示可能应用在这张表中的索引，一个或多个，但不一定实际使用到，这个表里面存在且可能会被使用的索引，可能会在这个字段下面出现，但是一般都以key为准。
7. key ：实际使用到的索引，如果为NULL，则没有使用索引。实际使用的索引，如果为null,则没有使用索引，否则会显示你使用了哪些索引，查询中若使用了覆盖索引（查询的列刚好是索引），则该索引仅出现在key列表。
8. key_len ：表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度
9. ref ：显示索引的哪一列被使用了，如果可能的话，是一个常数，哪些列或常量被用于查找索引列上的值
10. rows ：根据表统计信息及索引选用情况，大致估算出找到所需的记录所需读取的行数
11. filtered ：查询的表行占表的百分比 是查询的行数与总行数的比值。其实作用与rows差不多，都是数值越小，效率越高。
12. Extra ：包含不适合在其它列中显示但十分重要的额外信息
    * Using filesort： 表示当SQL中有一个地方需要对一些数据进行排序的时候，优化器找不到能够使用的索引，所以只能使用外部的索引排序，外部排序就不断的在磁盘和内存中交换数据，这样就摆脱不了很多次磁盘IO，以至于SQL执行的效率很低。反之呢？由于索引的底层是B+Tree实现的，他的叶子节点本来就是有序的，这样的查询能不爽吗？
    表示mysql会使用一个外部索引排序 而不是从表里的按照索引次序读到相关内容，可能在内存和磁盘上进行排序，mysq无法使用这个排序的缩影完成的排序叫做 using filesort
    * Using tempporary 表示在对MySQL查询结果进行排序时，使用了临时表,,这样的查询效率是比外部排序更低的，常见于order by和group by。
    * Using index 表示使用了索引，很优秀
    * Using where 使用了where但是好像没啥用。
    * Using join buffer 表明使用了连接缓存,比如说在查询的时候，多表join的次数非常多，那么将配置文件中的缓冲区的join buffer调大一些。
    * impossible where 筛选条件没能筛选出任何东西
    * distinct 优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作

3.修改sql或者尽量让sql走索引
---
