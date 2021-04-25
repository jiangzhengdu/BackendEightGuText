常用语句
===

 1、现有表以及数据添加字段

db.tbGoodsConsultant.update({}, {$set:{nFlagState:0}}, false, true);

2、给表字段添加索引

db.tbGoodsConsultant.ensureIndex({nFlagState:1});

3、增加数据

> db.food.save({"name":"jack","address":{"city":"Shanghai","post":021},"phone":[138,139]});
> db.food.save({"uid":"yushunzhi@sohu.com","AL":['test-1@sohu.com','test-2@sohu.com']});

4、删除表、数据库

> db.users.drop();
> db.dropDatabase();

5、创建索引、数字1表示升序 -1 表示降序

> db.user.ensureIndex({"lId":1,"name":-1});
> db.system.indexes.find();

6、删除索引

> db.mycoll.dropIndex(name)

7、去掉重复数据

> db.user.distinct('name');

8、排序sort 1:ASC -1:DESC

>db.user.find().sort({"age":1});

9、查询name中包含mongo的数据 %y%

> db.user.find({name:/y/});

10、查询name中以d开头的 like 'd%'

> db.user.find({name:/^d/});

11、查询指定列name、age数据（name也可以用true||false，true和name：1等同）

> db.user.find({},{name:1,age:1});

12、查询2条以后的数据

> db.user.find().skip(2);

13、查询在2-10之间的数据

> db.user.find().limit(10).skip(2);

14、or与查询 age=21 or age=22

> db.user.find({$or:[{age:21},{age:22}]});

15、相当于：update user set age = age + 2 , name = 'dylan_xu' where name='dylan';

> db.user.update({name:'dylan'},{$inc:{age:2},$set:{name:'dylan_xu'}},false,true);

16、advanced queries:高级查询

条件操作符
$gt : >
$lt : <
$gte: >=
$lte: <=
$ne : !=、<>
$in : in
$nin: not in
$all: all
$not: 反匹配(1.3.3及以上版本)
