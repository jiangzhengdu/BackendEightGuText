ReentrantLock
===

ReentantLock 继承接口 Lock 并实现了接口中定义的方法， 他是一种可重入锁， 除了能完
成 synchronized 所能完成的所有工作外，还提供了诸如可响应中断锁、可轮询锁请求、定时锁等 避免多线程死锁的方法。
Lock 接口的主要方法
void lock(): 执行此方法时, 如果锁处于空闲状态, 当前线程将获取到锁. 相反, 如果锁已经 被其他线程持有, 将禁用当前线程, 直到当前线程获取到锁.
boolean tryLock(): 如果锁可用, 则获取锁, 并立即返回 true, 否则返回 false. 该方法和 lock()的区别在于, tryLock()只是"试图"获取锁, 如果锁不可用, 不会导致当前线程被禁用, 当前线程仍然继续往下执行代码. 而 lock()方法则是一定要获取到锁, 如果锁不可用, 就一 直等待, 在未获得锁之前,当前线程并不继续向下执行.
void unlock():执行此方法时, 当前线程将释放持有的锁. 锁只能由持有者释放, 如果线程 并不持有锁, 却执行该方法, 可能导致异常的发生.
Condition newCondition(): 条件对象，获取等待通知组件。该组件和当前的锁绑定， 当前线程只有获取了锁，才能调用该组件的 await()方法，而调用后，当前线程将缩放锁。
getHoldCount() : 查询当前线程保持此锁的次数，也就是执行此线程执行 lock 方法的次 数。
getQueueLength() : 返回正等待获取此锁的线程估计数，比如启动 10 个线程， 1 个 线程获得锁，此时返回的是 9
getWaitQueueLength: (Condition condition)返回等待与此锁相关的给定条件的线 程估计数。比如 10 个线程，用同一个 condition 对象，并且此时这 10 个线程都执行了 condition 对象的 await 方法，那么此时执行此方法返回 10
hasWaiters(Condition condition): 查询是否有线程等待与此锁有关的给定条件 (condition)，对于指定 contidion 对象，有多少线程执行了 condition.await 方法
hasQueuedThread(Thread thread): 查询给定线程是否等待获取此锁
hasQueuedThreads(): 是否有线程等待此锁
isFair(): 该锁是否公平锁
isHeldByCurrentThread(): 当前线程是否保持锁锁定，线程的执行 lock 方法的前后分 别是 false 和 true
isLock(): 此锁是否有任意线程占用
lockInterruptibly() : 如果当前线程未被中断，获取锁 tryLock() : 尝试获得锁，仅在调用时锁未被线程占用，获得锁
tryLock(long timeout TimeUnit unit): 如果锁在给定等待时间内没有被另一个线程保持， 则获取该锁。

ReentrantLock源码分析
---

类的继承关系
----

ReentrantLock实现了Lock接口，Lock接口中定义了lock与unlock相关操作，并且还存在newCondition方法，表示生成一个条件。

类的内部类
----

ReentrantLock总共有三个内部类，并且三个内部类是紧密相关的，下面先看三个类的关系。

说明: ReentrantLock类内部总共存在Sync、NonfairSync、FairSync三个类，NonfairSync与FairSync类继承自Sync类，Sync类继承自AbstractQueuedSynchronizer抽象类。下面逐个进行分析。
