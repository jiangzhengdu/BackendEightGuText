抽象队列同步器
===

AQS 核心思想
---

AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。

AQS使用它使用一个**整型的volatile的int变量**（命名为state）来维护同步状态，来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用**CAS对该同步状态进行原子操作实现对其值的修改**。

> private volatile int state;//共享变量，使用volatile修饰保证线程可见性

抽象队列同步器（AbstractQueuedSynchronizer，简称AQS）是用来构建锁或者其他同步组件的基础框架，

维护一个state状态，尝试加锁的时候通过CAS修改值，如果成功设置为1，并且把当前线程ID赋值，则代表加锁成功，一旦获取到锁，则其他的线程
就会被阻塞进入到阻塞队列自旋，获取得到锁的线程释放锁的时候就会唤醒阻塞队列中的线程，释放锁的时候会把state重新设置为0，同时当前的
线程id设置为空

AQS 对资源的共享方式
---

AQS定义两种资源共享方式

Exclusive(独占)：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：

* 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
* 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
Share(共享)：多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。
ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护(如获取资源失败入队/唤醒出队等)，AQS已经在上层已经帮我们实现好了。

AbstractQueuedSynchronizer数据结构
---

AbstractQueuedSynchronizer类底层的数据结构是使用CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。其中Sync queue，即同步队列，是双向链表，包括head结点和tail结点，head结点主要用作后续的调度。而Condition queue不是必须的，其是一个单向链表，只有当使用Condition时，才会存在此单向链表。并且可能会有多个Condition queue。

类的属性属性中
---

包含了头节点head，尾结点tail，状态state、自旋时间spinForTimeoutThreshold，还有AbstractQueuedSynchronizer抽象的属性在内存中的偏移地址，通过该偏移地址，可以获取和设置该属性的值，同时还包括一个静态初始化块，用于加载内存偏移地址。

类的核心方法 - acquire方法
---

该方法以独占模式获取(资源)，忽略中断，即线程在aquire过程中，中断此线程是无效的。源码如下:

* 首先调用tryAcquire方法，调用此方法的线程会试图在独占模式下获取对象状态。此方法应该查询是否允许它在独占模式下获取对象状态，如果允许，则获取它。在AbstractQueuedSynchronizer源码中默认会抛出一个异常，即需要子类去重写此方法完成自己的逻辑。之后会进行分析。
* 若tryAcquire失败，则调用addWaiter方法，addWaiter方法完成的功能是将调用此方法的线程封装成为一个结点并放入Sync queue。
* 调用acquireQueued方法，此方法完成的功能是Sync queue中的结点不断尝试获取资源，若成功，则返回true，否则，返回false。
* 由于tryAcquire默认实现是抛出异常，所以此时，不进行分析，之后会结合一个例子进行分析。

addWaiter方法使用快速添加的方式往sync queue尾部添加结点，如果sync queue队列还没有初始化，则会使用enq插入队列中，enq方法源码如下

acquireQueue方法
---

首先获取当前节点的前驱节点，如果前驱节点是头节点并且能够获取(资源)，代表该当前节点能够占有锁，设置头节点为当前节点，返回。否则，调用shouldParkAfterFailedAcquire和parkAndCheckInterrupt方法只有当该节点的前驱结点的状态为SIGNAL时，才可以对该结点所封装的线程进行park操作。否则，将不能进行park操作。再看parkAndCheckInterrupt方法，源码如下
parkAndCheckInterrupt方法里的逻辑是首先执行park操作，即禁用当前线程，然后返回该线程是否已经被中断。再看final块中的cancelAcquire方法，其源码如下
该方法完成的功能就是取消当前线程对资源的获取，即设置该结点的状态为CANCELLED，接着我们再看unparkSuccessor方法，源码如下

现在，再来看acquireQueued方法的整个的逻辑。
逻辑如下:

1. 判断结点的前驱是否为head并且是否成功获取(资源)。
2. 若步骤1均满足，则设置结点为head，之后会判断是否finally模块，然后返回。
3. 若步骤2不满足，则判断是否需要park当前线程，是否需要park当前线程的逻辑是判断结点的前驱结点的状态是否为SIGNAL，若是，则park当前结点，否则，不进行park操作。
4. 若park了当前线程，之后某个线程对本线程unpark后，并且本线程也获得机会运行。那么，将会继续进行步骤①的判断。

类的核心方法 - release方法
---

其中，tryRelease的默认实现是抛出异常，需要具体的子类实现，如果tryRelease成功，那么如果头节点不为空并且头节点的状态不为0，则释放头节点的后继结点

ABA问题
---

使用AtomicStampedReference 加入了预期标志和更新标志两个字段
更新的时候不仅仅检查值，还要检查当前标志是否等于预期的标志，如果相等的话才会修改。
