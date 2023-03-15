ThreadLocal简介
===

我们在Java 并发 - 并发理论基础总结过线程安全(是指广义上的共享资源访问安全性，因为线程隔离是通过副本保证本线程访问资源安全性，它不保证线程之间还存在共享关系的狭义上的安全性)的解决思路：
互斥同步: synchronized 和 ReentrantLock
非阻塞同步: CAS, AtomicXXXX无同步方案:
栈封闭，本地存储(Thread Local)，可重入代码

总结而言：ThreadLocal是一个将在多线程中为每一个线程创建单独的变量副本的类; 当使用ThreadLocal来维护变量时, ThreadLocal会为每个线程创建单独的变量副本, 避免因多线程操作共享变量而导致的数据不一致的情况。
