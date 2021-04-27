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

id标明sql的执行顺序（越大越先执行）

type字段和Extra字段
index/all表示全表扫描，是最慢的，每次看到这两个要特别注意

extra:

可以获取到更为详细的信息：

出现以下两项表示mysql无法使用索引，应尽可能优化

using temportory 表示在mysql中对查询结果排序的时候，使用临时表，常见于使用group by order by
using filesort  表示mysql会使用一个外部索引排序 而不是从表里的按照索引次序读到相关内容，可能在
内存和磁盘上进行排序，mysq无法使用这个排序的缩影完成的排序叫做 using filesort

3.修改sql或者尽量让sql走索引
---


