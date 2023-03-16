GCD算法
===

求最大公约数
---

``` java
    public int gcd(int a, int b) {
        if (b == 0) return b;
        gcd(b, a % b);
    }

```
