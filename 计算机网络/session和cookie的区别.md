cookie和session的区别
====

1. cookie数据存放在客户的浏览器上,请求头里面，session数据放在服务器上。

2. cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗
考虑到安全应当使用session。

3. session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能
考虑到减轻服务器性能方面，应当使用COOKIE。

4. 单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。

cookie 和session的联系
----

session是通过cookie来工作的
session和cookie之间是通过$_COOKIE['PHPSESSID']来联系的，通过$_COOKIE['PHPSESSID']可以知道session的id，从而获取到其他的信息。
在购物网站中通常将用户加入购物车的商品联通session_id记录到数据库中，当用户再次访问是，
通过sessionid就可以查找到用户上次加入购物车的商品。因为sessionid是唯一的，记录到数据库中就可以根据这个查找。
