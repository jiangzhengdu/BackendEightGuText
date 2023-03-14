Semaphore 信号量
===

Semaphore 是一种基于计数的信号量。它可以设定一个阈值，基于此，多个线程竞争获取许可信号，做完自己的申请后归还，
超过阈值后，线程申请许可信号将会被阻塞。 Semaphore可以用来构建一些对象池，资源池之类的，比如数据库连接池
实现互斥锁(计数器为 1)

我们也可以创建计数为 1 的 Semaphore，将其作为一种类似互斥锁的机制，这也叫二元信号量， 表示两种互斥状态。
代码实现

``` java
// 创建一个计数阈值为 5 的信号量对象
// 只能 5 个线程同时访问
  Semaphore semp = new Semaphore(5); try { // 申请许可
  semp.acquire();
  try { // 业务逻辑
        } catch (Exception e) {
  } finally { // 释放许可
  semp.release(); }
```
