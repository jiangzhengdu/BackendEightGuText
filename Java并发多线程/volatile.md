volatile的用法
===

一旦一个共享变量(类的成员变量、类的静态成员变量)被volatile修饰之后，那么就具备了两层语义
---

1. 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的,
volatile关键字会强制将修改的值立即写入主存。

2. 禁止进行指令重排序。 volatile不是原子性操作什么叫保证部分有序性?

int
当程序执行到volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见;在其后面的操作肯定还没有进行;

由于flag变量为volatile变量，那么在进行指令重排序的过程的时候，不会将语句3放到语句1、语句2前 面，也不会讲语句3放到语句4、语句5后面。但是要注意语句1和语句2的顺序、语句4和语句5的顺序是 不作任何保证的。

使用 Volatile 一般用于 状态标记量和单例模式的双检锁

volatile的作用详解
---

防重排序
----

我们从一个最经典的例子来分析重排序问题。大家应该都很熟悉单例模式的实现，而在并发环境下的单例实现方式，我们通常可以采用双重检查加锁(DCL)的方式来实现。其源码如下：

``` java
public class Singleton {
    public static volatile Singleton singleton;
    /**
     * 构造函数私有，禁止外部实例化
     */
    private Singleton() {};
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

现在我们分析一下为什么要在变量singleton之间加上volatile关键字。要理解这个问题，先要了解对象的构造过程，实例化一个对象其实可以分为三个步骤：
> 分配内存空间。初始化对象。将内存空间的地址赋值给对应的引用。
但是由于操作系统可以对指令进行重排序，所以上面的过程也可能会变成如下过程：分配内存空间。将内存空间的地址赋值给对应的引用。初始化对象如果是这个流程，多线程环境下就可能将一个未初始化的对象引用暴露出来，从而导致不可预料的结果。因此，为了防止这个过程的重排序，我们需要将变量设置为volatile类型的变量。

实现可见性
----

可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——**线程工作内存**。volatile关键字能有效的解决这个问题，我们看下下面的例子，就可以知道其作用

``` java
public class TestVolatile {
    private static boolean stop = false;

    public static void main(String[] args) {
        // Thread-A
        new Thread("Thread A") {
            @Override
            public void run() {
                while (!stop) {
                }
                System.out.println(Thread.currentThread() + " stopped");
            }
        }.start();

        // Thread-main
        try {
            TimeUnit.SECONDS.sleep(1);
            System.out.println(Thread.currentThread() + " after 1 seconds");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        stop = true;
    }
}
```

执行输出如下

``` sh
Thread[main,5,main] after 1 seconds

// Thread A一直在loop, 因为Thread A 由于可见性原因看不到Thread Main 已经修改stop的值

```

可以看到 Thread-main 休眠1秒之后，设置 stop = ture，但是Thread A根本没停下来，这就是可见性问题。如果通过在stop变量前面加上volatile关键字则会真正stop:

``` sh
Thread[main,5,main] after 1 seconds
Thread[Thread A,5,main] stopped
```

保证原子性:单次读/写
----

volatile不能保证完全的原子性，只能保证单次的读/写操作具有原子性。先从如下两个问题来理解（后文再从内存屏障的角度理解）：

问题1： i++为什么不能保证原子性?
----

对于原子性，需要强调一点，也是大家容易误解的一点：对volatile变量的单次读/写操作可以保证原子性的，如long和double类型变量，但是并不能保证i++这种操作的原子性，因为本质上i++是读、写两次操作。现在我们就通过下列程序来演示一下这个问题：

``` java
public class VolatileTest01 {
    volatile int i;

    public void addI(){
        i++;
    }

    public static void main(String[] args) throws InterruptedException {
        final  VolatileTest01 test01 = new VolatileTest01();
        for (int n = 0; n < 1000; n++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test01.addI();
                }
            }).start();
        }
        Thread.sleep(10000);//等待10秒，保证上面程序执行完成
        System.out.println(test01.i);
    }
}
```

大家可能会误认为对变量i加上关键字volatile后，这段程序就是线程安全的。大家可以尝试运行上面的程序。下面是我本地运行的结果：981 可能每个人运行的结果不相同。不过应该能看出，volatile是无法保证原子性的(否则结果应该是1000)。
原因也很简单，i++其实是一个复合操作，包括三步骤：读取i的值。对i加1。将i的值写回内存。
volatile是无法保证这三个操作是具有原子性的，我们可以通过AtomicInteger或者Synchronized来保证+1操作的原子性。 注：上面几段代码中多处执行了Thread.sleep()方法，目的是为了增加并发问题的产生几率，无其他作用。

问题2： 共享的long和double变量的为什么要用volatile?
----

因为long和double两种数据类型的操作可分为高32位和低32位两部分，因此普通的long或double类型读/写可能不是原子的。因此，鼓励大家将共享的long和double变量设置为volatile类型，这样能保证任何情况下对long和double的单次读/写操作都具有原子性。

目前各种平台下的商用虚拟机都选择把 64 位数据的读写操作作为原子操作来对待，因此我们在编写代码时一般不把long 和 double 变量专门声明为 volatile多数情况下也是不会错的。

volatile 的实现原理
----

> volatile 变量的内存可见性是基于内存屏障(Memory Barrier)实现:

内存屏障，又称内存栅栏，是一个 CPU 指令。在程序运行时，为了提高执行性能，编译器和处理器会对指令进行重排序，JMM 为了保证在不同的编译器和 CPU 上有相同的结果，通过插入特定类型的内存屏障来禁止+ 特定类型的编译器重排序和处理器重排序，插入一条内存屏障会告诉编译器和 CPU：不管什么指令都不能和这条 Memory Barrier 指令重排序。

**lock 前缀的指令在多核处理器下会引发两件事情**:
将当前处理器缓存行的数据写回到系统内存。
写回内存的操作会使在其他 CPU 里缓存了该内存地址的数据无效。
为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存(L1，L2 或其他)后再进行操作，但操作完不知道何时会写到内存。
如果对声明了 volatile 的变量进行写操作，JVM 就会向处理器发送一条 lock 前缀的指令，将这个变量所在缓存行的数据写回到系统内存。
为了保证各个处理器的缓存是一致的，实现了缓存一致性协议(MESI)，每个处理器通过嗅探在**总线**上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。
所有**多核处理器**下还会完成：当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。volatile 变量通过这样的机制就使得每个线程都能获得该变量的最新值。

**lock指令**
----

Pentium 和早期的 IA-32 处理器中，lock 前缀会使处理器执行当前指令时产生一个 LOCK# 信号，会对总线进行锁定，其它 CPU 对内存的读写请求都会被阻塞，直到锁释放。 后来的处理器，加锁操作是由高速缓存锁代替总线锁来处理。 因为锁总线的开销比较大，锁总线期间其他 CPU 没法访问内存。 这种场景多缓存的数据一致通过缓存一致性协议(MESI)来保证

我们从文件名字 atomic_linux_x86.inline.hpp 中就能获取到一些信息，说明到现在位置已经跟具体的平台和具体的cpu的型号关系了，在x86平台上linux版本的实现，那么CAS是怎么实现的呢？
看这条指令__asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)" 不知道你明白没有，看到这我们可以下一条结论，原来在CPU层级有一条指令，叫cmpxchg。到这一步，你是不是觉得自己已经对于CAS已经理解的很透彻了呢？你有没有想过cmpxchg这条指令是原子操作吗？
前面的汇编我们忽略，你一定注意到了lock;1这是什么意思呢，从方法名字看lock if mp ，mp的全称是Multi Processor，多cpu，意思是什么呢，就是看你操作系统有多少个处理器，若果只有一个cpu一核的话就不需要原子性了，一定是顺序执行的，如果是多核心多cpu前面就要加lock，所以最红能够实现CAS的汇编指令就被我们揪出来了。最终的汇编指令是lock cmpxchg 指令，lock指令在执行后面指令的时候锁定一个北桥信号(锁定北桥信号比锁定总线轻量一些，感兴趣的自己百度)。
所以如果你是多核或者多个cpu，CPU在执行cmpxchg指令之前会执行lock锁定总线，实际是锁定北桥信号。我不释放这把锁谁也过不去，以此来保证cmpxchg的原子性

**缓存一致性**
----

mesi
缓存是分段(line)的，一个段对应一块存储空间，称之为缓存行，它是 CPU 缓存中可分配的最小存储单元，大小 32 字节、64 字节、128 字节不等，这与 CPU 架构有关，通常来说是 64 字节。 LOCK# 因为锁总线效率太低，因此使用了多组缓存。 为了使其行为看起来如同一组缓存那样。因而设计了 缓存一致性协议。 缓存一致性协议有多种，但是日常处理的大多数计算机设备都属于 " **嗅探(snooping)**" 协议。 所有内存的传输都发生在一条共享的总线上，而所有的处理器都能看到这条总线。 缓存本身是独立的，但是内存是共享资源，所有的内存访问都要经过**仲裁**(同一个指令周期中，只有一个 CPU 缓存可以读写内存)。 CPU 缓存不仅仅在做内存传输的时候才与总线打交道，而是不停在嗅探总线上发生的数据交换，跟踪其他缓存在做什么。 当**一个缓存代表它所属的处理器去读写内存时，其它处理器都会得到通知**，它们以此来使自己的缓存保持同步。 只要某个处理器写内存，其它处理器马上知道这块内存在它们的缓存段中已经失效。

volatile 有序性实现
----

volatile 的 happens-before 关系
-----

happens-before 规则中有一条是 volatile 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。

volatile 禁止重排序
-----

为了性能优化，JMM 在不改变正确语义的前提下，会允许编译器和处理器对指令序列进行重排序。JMM 提供了内存屏障阻止这种重排序。Java 编译器会在生成指令系列时在适当的位置会插入内存屏障指令来禁止特定类型的处理器重排序。JMM 会针对编译器制定 volatile 重排序规则表。

第一个操作               第二个操作
                    普通写/读     volatile读       volatile写
普通读写                                           No
volatile读           NO           NO              NO

volatile写                        NO              NO

" NO " 表示禁止重排序。
为了实现 volatile 内存语义时，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎是不可能的，为此，JMM 采取了保守的策略。
>在每个 volatile 写操作的前面插入一个 StoreStore 屏障。
>在每个 volatile 写操作的后面插入一个 StoreLoad 屏障。
>在每个 volatile 读操作的后面插入一个 LoadLoad 屏障。
>在每个 volatile 读操作的后面插入一个 LoadStore 屏障。

**volatile 写是在前面和后面分别插入内存屏障，而 volatile 读操作是在后面插入两个内存屏障。**

StoreStore 屏障禁止上面的普通写和下面的 volatile 写重排序。
StoreLoad 屏障防止上面的 volatile 写与下面可能有的 volatile 读/写重排序。
LoadLoad 屏障禁止下面所有的普通读操作和上面的 volatile 读重排序。
LoadStore 屏障禁止下面所有的普通写操作和上面的 volatile 读重排序

volatile 的应用场景使用
----

volatile 必须具备的条件对变量的写操作不依赖于当前值。
该变量没有包含在具有其他变量的不变式中。
只有在状态真正独立于程序内其他内容时才能使用 volatile。

模式1：状态标志
-----

也许实现 volatile 变量的规范使用仅仅是使用一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或请求停机。

模式2：一次性安全发布(one-time safe publication)
-----

缺乏同步会导致无法实现可见性，这使得确定何时写入对象引用而不是原始值变得更加困难。在缺乏同步的情况下，可能会遇到某个对象引用的更新值(由另一个线程写入)和该对象状态的旧值同时存在。(这就是造成著名的双重检查锁定(double-checked-locking)问题的根源，其中对象引用在没有同步的情况下进行读操作，产生的问题是您可能会看到一个更新的引用，但是仍然会通过该引用看到不完全构造的对象)。

模式3：独立观察(independent observation)
-----

安全使用 volatile 的另一种简单模式是定期 发布 观察结果供程序内部使用。
例如，假设有一种环境传感器能够感觉环境温度。一个后台线程可能会每隔几秒读取一次该传感器，并更新包含当前文档的 volatile 变量。然后，其他线程可以读取这个变量，从而随时能够看到最新的温度值。

模式4：volatile bean 模式
-----

在 volatile bean 模式中，JavaBean 的所有数据成员都是 volatile 类型的，并且 getter 和 setter 方法必须非常普通 —— 除了获取或设置相应的属性外，不能包含任何逻辑。此外，对于对象引用的数据成员，引用的对象必须是有效不可变的。(这将禁止具有数组值的属性，因为当数组引用被声明为 volatile 时，只有引用而不是数组本身具有 volatile 语义)。对于任何 volatile 变量，不变式或约束都不能包含 JavaBean 属性。

模式5：开销较低的读－写锁策略volatile 的功能还不足以实现计数器
-----

因为 ++x 实际上是三种操作(读、添加、存储)的简单组合，如果多个线程凑巧试图同时对 volatile 计数器执行增量操作，那么它的更新值有可能会丢失。 如果读操作远远超过写操作，可以结合使用内部锁和 volatile 变量来减少公共代码路径的开销。 安全的计数器使用 synchronized 确保增量操作是原子的，并使用 volatile 保证当前结果的可见性。如果更新不频繁的话，该方法可实现更好的性能，因为读路径的开销仅仅涉及 volatile 读操作，这通常要优于一个无竞争的锁获取的开销。

模式6：双重检查(double-checked)
-----

就是我们上文举的例子。单例模式的一种实现方式，但很多人会忽略 volatile 关键字，因为没有该关键字，程序也可以很好的运行，只不过代码的稳定性总不是 100%，说不定在未来的某个时刻，隐藏的 bug 就出来了
