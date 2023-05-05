剑指 Offer 05替换空格
===

题目
---

请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

思路
---

使用Stringbuilder，遍历字符串，遇到空格就添加`%20`，否则添加当前字符。

代码
---

```java
class Solution {
    public String replaceSpace(String s) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) != ' ') {
                sb.append(s.charAt(i));
            }
            else {
                sb.append("%20");
            }
        }
        return sb.toString();
            
    }
}
```
