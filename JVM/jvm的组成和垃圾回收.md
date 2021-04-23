jvm是由五个部分组成的
===

线程私有的
---

1. 程序计数器  pc：是当前线程所执行的字节码的行号指示器
2. java虚拟机栈   JVM stack：存放的是栈帧 有局部变量 操作栈 动态链接 函数的引用 函数的出口 对运行时常量池的引用
3. 本地方法栈  native method area 存放的是不是java写的方法

线程共有的
---

1. 方法区/永久代/元数据 ：存放的加载的类的信息 常量 静态变量 运行的常量池
及编译后的代码等数据
在jdk8之后，被元空间取代，元空间不在虚拟机中，而使用本地内存。

2. 堆  
分为 新生代和老年代

新生代：（1/3）  8:1:1  主要使用copy算法
Eden  新对象的出身地，如果对象很大，则直接放到老年代，所以经常会触发minor GC
from survivor  to survivor

minor gc 采用的是复制算法
1.首先将eden区域和from区域进行复制到to，然后年龄加1。如果太老的话放到老年代
2.清空from和eden区域
3to和from区域互相换一下

老年代：（2/3）  主要使用mark-sweep算法
用于存放程序中生命周期比较长的对象
Major GC
一般在minor GC之后，在老年区放不下了，才触发

mark-sweep标记清除算法：碎片化严重
copying算法：针对标记清除算法，一次只使用一半的内存，满了就会把存活下来的复制过去，问题是只用了一半
标记-整理算法  mark-compact算法

gc的过程：
而老年代因为每次只回收少量对象，因而采用 Mark-Compact 算法。

1. JAVA 虚拟机提到过的处于方法区的永生代(Permanet Generation)， 它用来存储 class 类，常量，方法描述等。
对永生代的回收主要包括废弃常量和无用的类。
2. 对象的内存分配主要在新生代的 Eden Space 和 Survivor Space 的 From Space(Survivor 目前存 放对象的那一块)，少数情况会直接分配到老生代。
3. 当新生代的 Eden Space 和 From Space 空间不足时就会发生一次 GC，进行 GC 后，EdenSpace 和 From Space 区的存活对象会被挪到 To Space，然后将Eden Space 和 FromSpace 进行清理。
4. 如果 To Space 无法足够存储某个对象，则将这个对象存储到老生代。
5. 在进行 GC 后，使用的便是 Eden Space 和 To Space 了，如此反复循环。
6. 当对象在 Survivor 区躲过一次 GC 后，其年龄就会+1。 默认情况下年龄到达15的对象会被移到老生代中。

JAVA的引用
---

JAVA 强引用
在 Java 中最常见的就是强引用， 把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，
它处于可达状态，它是不可能被垃圾回收机制回收的，即使该对象以后永远都不会被用到，JVM也不会回收。
因此强引用是造成 Java内存泄漏的主要原因之一。

JAVA软引用
软引用需要用 SoftReference 类来实现，对于只有软引用的对象来说，
当系统内存足够时它不会被回收，当系统内存空间不足时它会被回收。
软引用通常用在对内存敏感的程序中。

JAVA弱引用
只要是一GC，就会被回收

java虚引用
不单独使用，和队列一起使用，主要目的是跟踪对象被回收的状态

GC roots:

a.虚拟机栈(栈桢中的本地变量表)中的引用的对象

b.方法区中的类静态属性引用的对象

c.方法区中的常量引用的对象

d.本地方法栈中JNI的引用的对象

Serial算法： stop the world  采用复制算法，是最简单的，默认的新生代回收

Parnew算法：是多线程的

CMS  concurrent mark sweep  目的是获取最少的垃圾停顿的时间
分为：
初时标记： 需要stop the world
并发标记：
重新标记： stop the world
并发清除：

由于第二个和第四个是时间最长的，所以很快

G1 收集器：
garage first：

是基于mark-sweep算法的
可以精确控制停顿时间，不牺牲吞吐量的情况下，实现低停顿回收  
避免垃圾全局垃圾回收，把堆内存分为几个不同的区域，对每个区域的垃圾进行跟踪，
选出垃圾最多的区域进行回收
区域划分和优先级回收 可以保证更好的垃圾回收效率

什么时候会出现FullGC
---

1. 老年代空间不足 旧生代空间只有在新生代对象转入及创建为大对象、大数组时才会出现不足的现象，当执行Full GC后空间仍然不足，则抛出如下错误:
java.lang.OutOfMemoryError: Java heap space 为避免以上两种状况引起的FullGC，
调优时应尽量做到让对象在Minor GC阶段被回收、让对象在新生代多存活一段时间及不要创建过大的对象及数组。

2. 方法区/永久代（Permanet Generation）空间满 PermanetGeneration中存放的为一些class的信息等，当系统中要加载的类、反射的类和调用的 方法较多时，Permanet Generation可能会被占满，在未配置为采用CMS GC的情况下会执行Full GC。如果经过Full GC仍然回收不了，那么JVM会抛出如下错误信息: java.lang.OutOfMemoryError: PermGen space
为避免Perm Gen占满造成Full GC现象，可采用的方法为增大PermGen空间或转为使用CMS GC。

3. CMS GC时出现promotion failed和concurrent mode failure 对于采用CMS进行旧生代GC的程序而言，尤其要注意GC日志中是否有promotion failed和 concurrent mode failure两种状况，当这两种状况出现时可能会触发Full GC。
promotionfailed是在进行Minor GC时，survivor space放不下、对象只能放入旧生代，而此时旧 生代也放不下造成的;concurrent mode failure是在执行CMS GC的过程中同时有对象要放入旧生 代，而此时旧生代空间不足造成的。
应对措施为:增大survivorspace、旧生代空间或调低触发并发GC的比率，但在JDK 5.0+、6.0+的 版本中有可能会由于JDK的bug29导致CMS在remark完毕后很久才触发sweeping动作。对于这种 状况，可通过设置-XX:CMSMaxAbortablePrecleanTime=5(单位为ms)来避免。

4. 统计得到的Minor GC晋升到旧生代的平均大小大于旧生代的剩余空间 这是一个较为复杂的触发情况，Hotspot为了避免由于新生代对象晋升到旧生代导致旧生代空间不 足的现象，在进行Minor GC时，做了一个判断，如果之前统计所得到的Minor GC晋升到旧生代的 平均大小大于旧生代的剩余空间，那么就直接触发Full GC。 例如程序第一次触发MinorGC后，有6MB的对象晋升到旧生代，那么当下一次Minor GC发生时， 首先检查旧生代的剩余空间是否大于6MB，如果小于6MB，则执行Full GC。 当新生代采用PSGC时，方式稍有不同，PS GC是在Minor GC后也会检查，例如上面的例子中第一 次Minor GC后，PS GC会检查此时旧生代的剩余空间是否大于6MB，如小于，则触发对旧生代的 回收。除了以上4种状况外，对于使用RMI来进行RPC或管理的Sun JDK应用而言，默认情况下会一 小时执行一次Full GC。可通过在启动时通过- java-Dsun.rmi.dgc.client.gcInterval=3600000来设 置Full GC执行的间隔时间或通过-XX:+ DisableExplicitGC来禁止RMI调用System.gc
