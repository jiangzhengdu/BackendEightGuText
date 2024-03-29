# **MD的基本语法**

## 1. **Headers**

![标题](../Picture/MD学习/标题.jpg)
<!-- # 一级标题

## 二级标题

### 三级标题

#### 四级标题

##### 五级标题

###### 六级标题

####### 7就不行了 -->

## 2. **Links**

### Inline Link (行内式)

[Google](http://www.google.com)

> 就是链接的文字显示要放在[]中，链接的地址放到()中即可

### Reference Link (参数式)

[Baidu]: http://www.baidu.com
[Google]: http://www.google.com

>例子 this is [百度][Baidu] ,this is [谷歌][Google]

## 3. **Images**

### 本地图片（相对路径）

![标题](../Picture/MD学习/本地图片.jpg)

### 网络图片

![Emma][emma]

> 总结一下：图片和链接的用法基本一样，只是图片需要写一个！

[emma]: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201903%2F09%2F20190309104419_3GRKk.jpeg&refer=http%3A%2F%2Fb-ssl.duitang.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1637137316&t=21315da1155909024886f52c8326e004

## **4. Italics and Bold**

比如

> _this is italics_ (surround words with an underscore(_))  

比如

> **this is bold** (surround words with two asterisks(**))

比如

> **_this is combination_** you can use _both italics and bold_ in the same line

## **5. Blockquotes**

>to create a block quote ,all you have to do is perface a line with the "greater than " create (>)

## **6. List**

> To create an unordered list, you'll want to preface each item in the list with an asterisk ( * ). Each list item also gets its own line.

* list 1
* list 2
* list 3

> An ordered list is prefaced with numbers, instead of asterisks. Take a look at this recipe:

1. **this is 1**
2. this is 2
3. this is 3

how to make a list with more deptgh or to nest one list within another.
All you have to do is to remember to indent each asterisk one space more than the preceding item.

* this is list mother
  * this is list son
* this is list  father
  * this is list son
  * son

## **7. Paragraphs**  

> 通过每一行后面加上两个space，然后就是一段  
这就是一段

否则就不是一段

> 注意： softbreak就是加两个space，hardbreak就是加一个空行

## **8. Code**

如果代码量比较少的情况，只有一行，则使用（`)反单引号包起来

` int a = 10; `

如果是多行的话，使用三个反引号(```),注释可以在三个反引号之后写

``` java
public static void main(Sting[] argv){
    System.out.println("hello world!");
}
```

## **9. 删除**

>删除线使用两个~包起来  
~~因该删除~~

## **10. Table**

|name| age | sex |
|:---:|:---:|:---:|
|du | 20 | male|
|yun| 20 | female|

>分割线后面的冒号表示对齐方式，写在左边表示左对齐，右边为右对齐，两边都写表示居中

### 参考资料

> [参考资料1](https://www.markdowntutorial.com/lesson/6/)  
[参考资料2](https://www.cnblogs.com/liugang-vip/p/6337580.html)
