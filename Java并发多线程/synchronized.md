关键字: synchronized详解
===

在C程序代码中我们可以利用操作系统提供的互斥锁来实现同步块的互斥访问及线程的阻塞及唤醒等工作。在Java中除了提供Lock API外还在语法层面上提供了synchronized关键字来实现互斥同步原语, 本文将对synchronized关键字详细分析。

Synchronized的使用
---

在应用Sychronized关键字时需要把握如下注意点：一把锁只能同时被一个线程获取，没有获得锁的线程只能等待；
每个实例都对应有自己的一把锁(this),不同实例之间互不影响；例外：锁对象是*.class以及synchronized修饰的是static方法的时候，所有对象公用同一把锁
synchronized修饰的方法，无论方法正常执行完毕还是抛出异常，都会释放锁

对象锁
----

包括方法锁(默认锁对象为this,当前实例对象)和同步代码块锁(自己指定锁对象)

``` Java
public class SynchronizedObjectLock implements Runnable {
    static SynchronizedObjectLock instance = new SynchronizedObjectLock();

    @Override
    public void run() {
        // 同步代码块形式——锁为this,两个线程使用的锁是一样的,线程1必须要等到线程0释放了该锁后，才能执行
        synchronized (this) {
            System.out.println("我是线程" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "结束");
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
    }
}

``` Java
public class SynchronizedObjectLock implements Runnable {
    static SynchronizedObjectLock instance = new SynchronizedObjectLock();
    // 创建2把锁
    Object block1 = new Object();
    Object block2 = new Object();

    @Override
    public void run() {
        // 这个代码块使用的是第一把锁，当他释放后，后面的代码块由于使用的是第二把锁，因此可以马上执行
        synchronized (block1) {
            System.out.println("block1锁,我是线程" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("block1锁,"+Thread.currentThread().getName() + "结束");
        }

        synchronized (block2) {
            System.out.println("block2锁,我是线程" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("block2锁,"+Thread.currentThread().getName() + "结束");
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
    }
}
```

方法锁形式：synchronized修饰普通方法，锁对象默认为this
----

类锁
---

指synchronize修饰静态的方法或指定锁对象为Class对象

``` java

public class SynchronizedObjectLock implements Runnable {
    static SynchronizedObjectLock instance1 = new SynchronizedObjectLock();
    static SynchronizedObjectLock instance2 = new SynchronizedObjectLock();

    @Override
    public void run() {
        method();
    }

    // synchronized用在静态方法上，默认的锁就是当前所在的Class类，所以无论是哪个线程访问它，需要的锁都只有一把
    public static synchronized void method() {
        System.out.println("我是线程" + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "结束");
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(instance1);
        Thread t2 = new Thread(instance2);
        t1.start();
        t2.start();
    }
}

```

synchronized指定锁对象为Class对象
----

``` java
public class SynchronizedObjectLock implements Runnable {
    static SynchronizedObjectLock instance1 = new SynchronizedObjectLock();
    static SynchronizedObjectLock instance2 = new SynchronizedObjectLock();

    @Override
    public void run() {
        // 所有线程需要的锁都是同一把
        synchronized(SynchronizedObjectLock.class){
            System.out.println("我是线程" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "结束");
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(instance1);
        Thread t2 = new Thread(instance2);
        t1.start();
        t2.start();
    }
}

```

Synchronized原理分析
---

现象、时机(内置锁this)、深入JVM看字节码(反编译看monitor指令)

Synchronized 作用范围
---

1. 作用于方法时，锁住的是对象的实例(this);
2. 当作用于静态方法时，锁住的是Class实例，又因为Class的相关数据存储在永久带
PermGen(jdk1.8 则是 metaspace)，永久带是全局共享的，因此静态方法锁相当于类的一个全
局锁，会锁所有调用该方法的线程;
3. synchronized 作用于一个对象实例时，锁住的是所有以该对象为锁的代码块。 它有多个队列，当
多个线程一起访问某个对象监视器的时候，对象监视器会将这些线程存储在不同的容器中。

Synchronized 核心组件
---

1 Wait Set: 哪些调用 wait 方法被阻塞的线程被放置在这里。
2 Contention List: 竞争队列，所有请求锁的线程首先被放在这个竞争队列中;
3 Entry List: Contention List 中那些有资格成为候选资源的线程被移动到 Entry List 中;
4 OnDeck:任意时刻， 最多只有一个线程正在竞争锁资源，该线程被成为 OnDeck;
5 Owner:当前已经获取到所资源的线程被称为 Owner;
6 !Owner:当前释放锁的线程。

Synchronized 实现
---

1. JVM 每次从队列的尾部取出一个数据用于锁竞争候选者(OnDeck)，但是并发情况下，
ContentionList 会被大量的并发线程进行 CAS 访问，为了降低对尾部元素的竞争， JVM 会将 一部分线程移动到 EntryList 中作为候选竞争线程。
2. Owner 线程会在 unlock 时，将 ContentionList 中的部分线程迁移到 EntryList 中，并指定 EntryList 中的某个线程为 OnDeck 线程(一般是最先进去的那个线程)。
3. Owner 线程并不直接把锁传递给 OnDeck 线程，而是把锁竞争的权利交给 OnDeck， OnDeck 需要重新竞争锁。
这样虽然牺牲了一些公平性，但是能极大的提升系统的吞吐量，在 JVM 中，也把这种选择行为称之为“竞争切换”。
4. OnDeck 线程获取到锁资源后会变为 Owner 线程，而没有得到锁资源的仍然停留在 EntryList 中。
如果 Owner 线程被 wait 方法阻塞，则转移到 WaitSet 队列中，直到某个时刻通过 notify 或者 notifyAll 唤醒，会重新进去 EntryList 中。
5. 处于 ContentionList、 EntryList、 WaitSet 中的线程都处于阻塞状态，该阻塞是由操作系统来完成的(
  Linux 内核下采用 pthread_mutex_lock 内核函数实现的)。
6. Synchronized 是非公平锁。 Synchronized 在线程进入 ContentionList 时， 等待的线程会先尝试自旋获取锁，
如果获取不到就进入 ContentionList，这明显对于已经进入队列的线程是不公平的，还有一个不公平的事情就是自旋获取锁的线程还可能直接抢占 OnDeck 线程的锁 资源。
7. 每个对象都有个 monitor 对象， 加锁就是在竞争 monitor 对象，代码块加锁是在前后分别加
上 monitorenter 和 monitorexit 指令来实现的，方法加锁是通过一个标记位来判断的
8. synchronized 是一个重量级操作，需要调用操作系统相关接口，性能是低效的，有可能给线
 程加锁消耗的时间比有用操作消耗的时间更多。

9. Java1.6， synchronized 进行了很多的优化， 有适应自旋、锁消除、锁粗化、轻量级锁及偏向锁等，
效率有了本质上的提高。在之后推出的 Java1.7 与 1.8 中，均对该关键字的实现机理做了优化。引入了偏向锁和轻量级锁。
都是在对象头中有标记位，不需要经过操作系统加锁。
10. 锁可以从偏向锁升级到轻量级锁，再升级到重量级锁。这种升级过程叫做锁膨胀;
11. JDK 1.6 中默认是开启偏向锁和轻量级锁，可以通过-XX:-UseBiasedLocking 来禁用偏向锁。




Synchronized 同步锁
synchronized 它可以把任意一个非 NULL 的对象当作锁。 他属于独占式的悲观锁，同时属于可重入锁。

