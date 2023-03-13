转换
===

如何实现数组和 List 之间的转换？
---

``` java
// list to array
List<String> list = new ArrayList<String>();
list.add("123");
list.add("456");
list.toArray();
// array to list
String[] array = new String[]{"123","456"};
Arrays.asList(array);
```
