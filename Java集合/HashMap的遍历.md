HashMap的遍历
===

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

