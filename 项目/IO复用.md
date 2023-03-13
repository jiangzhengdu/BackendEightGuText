Linux IO模型：阻塞/非阻塞/IO复用 同步/异步 Select/Epoll/AIO
===

IO概念
Linux的内核将所有外部设备都可以看做一个文件来操作。那么我们对与外部设备的操作都可以看做对文件进行操作。我们对一个文件的读写，都通过调用内核提供的系统调用；内核给我们返回一个file descriptor（fd,文件描述符）。而对一个socket的读写也会有相应的描述符，称为socketfd(socket描述符）。描述符就是一个数字，指向内核中一个结构体（文件路径，数据区，等一些属性）。那么我们的应用程序对文件的读写就通过对描述符的读写完成。

linux将内存分为内核区，用户区。linux内核给我们管理所有的硬件资源，应用程序通过调用系统调用和内核交互，达到使用硬件资源的目的。应用程序通过系统调用read发起一个读操作，这时候内核创建一个文件描述符，并通过驱动程序向硬件发送读指令，并将读的的数据放在这个描述符对应结构体的内核缓存区中，然后再把这个数据读到用户进程空间中，这样完成了一次读操作；
但是大家都知道I/O设备相比cpu的速度是极慢的。linux提供的read系统调用，也是一个阻塞函数。这样我们的应用进程在发起read系统调用时，就必须阻塞，就进程被挂起而等待文件描述符的读就绪，那么什么是文件描述符读就绪，什么是写就绪？

读就绪：就是这个文件描述符的接收缓冲区中的数据字节数大于等于套接字接收缓冲区低水位标记的当前大小；

写就绪：该描述符发送缓冲区的可用空间字节数大于等于描述符发送缓冲区低水位标记的当前大小。（如果是socket fd，说明上一个数据已经发送完成）。

接收低水位标记和发送低水位标记：由应用程序指定，比如应用程序指定接收低水位为64个字节。那么接收缓冲区有64个字节，才算fd读就绪；

综上所述，一个基本的IO，它会涉及到两个系统对象，一个是调用这个IO的进程对象，另一个就是系统内核(kernel)。当一个read操作发生时，它会经历两个阶段：

    通过read系统调用想内核发起读请求。
    内核向硬件发送读指令，并等待读就绪。
    内核把将要读取的数据复制到描述符所指向的内核缓存区中。
    将数据从内核缓存区拷贝到用户进程空间中。

IO模型
---

在linux系统下面，根据IO操作的是否被阻塞以及同步异步问题进行分类，可以得到下面五种IO模型：

1. 阻塞I/O模型
最流行的I/O模型是阻塞I/O模型，缺省情形下，所有文件操作都是阻塞的。我们以套接口为例来讲解此模型。在进程空间中调用recvfrom，其系统调用直到数据报到达且被拷贝到应用进程的缓冲区中或者发生错误才返回，期间一直在等待。我们就说进程在从调用recvfrom开始到它返回的整段时间内是被阻塞的。

2. 非阻塞I/O模型
进程把一个套接口设置成非阻塞是在通知内核：当所请求的I/O操作不能满足要求时候，不把本进程投入睡眠，而是返回一个错误。也就是说当数据没有到达时并不等待，而是以一个错误返回。

3. I/O复用模型
linux提供select/poll，进程通过将一个或多个fd传递给select或poll系统调用，阻塞在select;这样select/poll可以帮我们侦测许多fd是否就绪。但是select/poll是顺序扫描fd是否就绪，而且支持的fd数量有限。linux还提供了一个epoll系统调用，epoll是基于事件驱动方式，而不是顺序扫描,当有fd就绪时，立即回调函数rollback；

4. 信号驱动异步I/O模型
首先开启套接口信号驱动I/O功能, 并通过系统调用sigaction安装一个信号处理函数（此系统调用立即返回，进程继续工作，它是非阻塞的）。当数据报准备好被读时，就为该进程生成一个SIGIO信号。随即可以在信号处理程序中调用recvfrom来读数据报，井通知主循环数据已准备好被处理中。也可以通知主循环，让它来读数据报。

5. 异步I/O模型
告知内核启动某个操作，并让内核在整个操作完成后(包括将数据从内核拷贝到用户自己的缓冲区)通知我们。这种模型与信号驱动模型的主要区别是：信号驱动I/O：由内核通知我们何时可以启动一个I/O操作；异步I/O模型：由内核通知我们I/O操作何时完成。

非阻塞IO详解
---

通过上面，我们知道，所有的IO操作在默认情况下，都是属于阻塞IO。尽管上图中所示的反复请求的非阻塞IO的效率底下（需要反复在用户空间和进程空间切换和判断，把一个原本属于IO密集的操作变为IO密集和计算密集的操作），但是在后面IO复用中，需要把IO的操作设置为非阻塞的，此时程序将会阻塞在select和poll系统调用中。把一个IO设置为非阻塞IO有两种方式：在创建文件描述符时，指定该文件描述符的操作为非阻塞；在创建文件描述符以后，调用fcntl()函数设置相应的文件描述符为非阻塞

创建描述符时，利用open函数和socket函数的标志设置返回的fd/socket描述符为O_NONBLOCK。

``` C
int sd=socket(int domain, int type|O_NONBLOCK, int protocol);
int fd=open(const char *pathname, int flags|O_NONBLOCK);
```

创建描述符后，通过调用fcntl函数设置描述符的属性为O_NONBLOCK

``` C
#include <unistd.h>
#include <fcntl.h> 
int fcntl(int fd, int cmd, ... /* arg */ ); 
  //例子
  if (fcntl(fd, F_SETFL, fcntl(sockfd, F_GETFL, 0)|O_NONBLOCK) == -1) {       
     return -1;
  } 
  return 0;
}
```

IO复用详解
---

在IO编程过程中，当需要处理多个请求的时，可以使用多线程和IO复用的方式进行处理。上面的图介绍了整个IO复用的过程，它通过把多个IO的阻塞复用到一个select之类的阻塞上，从而使得系统在单线程的情况下同时支持处理多个请求。和多线程/进程比较，I/O多路复用的最大优势是系统开销小，系统不需要建立新的进程或者线程，也不必维护这些线程和进程。IO复用常见的应用场景：

1. 客户程序需要同时处理交互式的输入和服务器之间的网络连接。
2. 客户端需要对多个网络连接作出反应。
3. 服务器需要同时处理多个处于监听状态和多个连接状态的套接字
4. 服务器需要处理多种网络协议的套接字。

目前支持I/O复用的系统调用有select、pselect、poll、epoll，下面几小结分别来学习一下select和epoll的使用。
Linux 2.6之前是select、poll，2.6之后是epoll，Windows是IOCP。

select
-----

``` C
#include <sys/select.h> 
int select(int maxfdps, fd_set *readfds, fd_set *writefds,fd_set *exceptfds, struct timeval *timeout); 
struct timeval{
  long tv_sec; //秒
  long tv_usec; //微秒
};
void FD_CLR(int fd, fd_set *set); //将一个给定的文件描述符从集合中删除
int  FD_ISSET(int fd, fd_set *set);  // 检查集合中指定的文件描述符是否可以读写 ?
void FD_SET(int fd, fd_set *set); //将一个给定的文件描述符加入集合之中
void FD_ZERO(fd_set *set);//清空集合
```

struct timeval *time结构体告知内核等待所指定描述字中的任何一个就绪可花多少时间。参数取值：

1. struct timeval * 0：永远等待下去，仅在有一个描述字准备好I/O时才返回
2. struct timeval *time：在有一个描述字准备好I/O时返回，但是不超过由该参数所指向的timeval结构中指定的秒数和微秒数。如果超过时间，没有描述字准备好，那就返回0。如果秒=微秒=0，检查描述字后立即返回，此时相当于轮询。
中间的三个参数readset、writeset和exceptset指定我们要让内核测试读、写和异常条件的描述字。如果我们对某一个的条件不感兴趣，就可以把它设为空指针。fd_set可以理解为一个集合，这个集合中存放的是文件描述符，可通过上面的四个宏进行设置（注意，fd_set不是struct fd_set。。。 刚刚开始调试程序就犯错误，返回storage size of 'fds' isn't known）。
第一个参数maxfdp1指定待测试的描述字个数，它的值是待测试的最大描述字加1（因此我们把该参数命名为maxfdp1），描述字0、1、2...maxfdp1-1均将被测试。

pselect
----

``` C
#include <sys/select.h> 
int pselect(int nfds, fd_set *readfds, fd_set *writefds,fd_set *exceptfds, const struct timespec *timeout,   const sigset_t *sigmask);
struct timespec{    
  time_t tv_sec; //秒    
  long tv_nsec; //纳秒
};
```

比较select和pselect函数，我们发现在原型上面有两个不同：

1. pselect使用timespec结构，而不使用timeval结构。timespec结构是POSIX的又一个发明。 这两个结构的区别在于第二个成员：新结构的该成员tv_nsec指定纳秒数，而旧结构的该成员tv_usec指定微秒数。
2. pselect函数增加了第六个参数：一个指向信号掩码的指针。该参数允许程序先禁止递交某些信号，再测试由这些当前被禁止的信号处理函数设置的全局变量，然后调用pselect，告诉它重新设置信号掩码。

首先我们看看信号掩码的概念：在POSIX下，每个进程有一个信号掩码(signal mask)。简单地说，信号掩码是一个“位图”，其中每一位都对应着一种信号。如果位图中的某一位为1，就表示在执行当前信号的处理程序期间相应的信号暂时被“屏蔽”，使得在执行的过程中不会嵌套地响应那种信号。为什么对某一信号进行屏蔽呢？我们来看一下对CTRL_C的处理。大家知道，当一个程序正在运行时，在键盘上按一下CTRL_C，内核就会向相应的进程发出一个SIGINT 信号，而对这个信号的默认操作就是通过do_exit()结束该进程的运行。但是，有些应用程序可能对CTRL_C有自己的处理，所以就要为SIGINT另行设置一个处理程序，使它指向应用程序中的一个函数，在那个函数中对CTRL_C这个事件作出响应。但是，在实践中却发现，两次CTRL_C事件往往过于密集，有时候刚刚进入第一个信号的处理程序，第二个SIGINT信号就到达了，而第二个信号的默认操作是杀死进程，这样，第一个信号的处理程序根本没有执行完。为了避免这种情况的出现，就在执行一个信号处理程序的过程中将该种信号自动屏蔽掉。所谓“屏蔽”，与将信号忽略是不同的，它只是将信号暂时“遮盖”一下，一旦屏蔽去掉，已到达的信号又继续得到处理。有关信号相关的知识参考我另外一个文章：《linux基础编程：进程通信之信号》。在我们开发过程中，如果进程阻塞与select函数，此时该阻塞被信号所打断，select返回-1，errno=EINTR的错误。由于这个原因，我们才有了pselect函数在信号上的优化处理。因此在信号和select都被使用的系统里，pselect有其作用。否则和select没有任何区别。

poll
----

``` cpp
#include <poll.h> 
int poll(struct pollfd *fds, nfds_t nfds, int timeout); 
struct pollfd { 
  int fd; /* file descriptor */ 
  short events; /* requested events to watch */ 
  short revents; /* returned events witnessed */    
}; 

```

和select()不一样，poll()没有使用低效的三个基于位的文件描述符set，而是采用了一个单独的结构体pollfd数组，由fds指针指向这个数组的第一个元素，其中nfds表示该数组的大小。每一个pollfd 结构体指定了一个被监视的文件描述符，每个结构体的events域是监视该文件描述符的事件掩码，由用户来设置这个域。revents域是文件描述符的操作结果事件掩码。内核在调用返回时设置这个域。events域中请求的任何事件都可能在revents域中返回，而部分事件不能出现在events中，详细如下表。

epoll
-----

在linux的网络编程中，很长的一段时间都在使用select来做事件触发。然而select逐渐暴露出了一些缺陷，使得linux不得不在新的内核中寻找出替代方案，那就是epoll。其实，epoll与select原理类似，只不过，epoll作出了一些重大改进，即：

1. 支持一个进程打开大数目的socket描述符(FD)
select 最不能忍受的是一个进程所打开的FD是有一定限制的，由FD_SETSIZE设置，默认值是2048。对于那些需要支持的上万连接数目的IM服务器来说显然太少了。这时候你一是可以选择修改这个宏然后重新编译内核，不过资料也同时指出这样会带来网络效率的下降，
二是可以选择多进程的解决方案(传统的 Apache方案)，不过虽然linux上面创建进程的代价比较小，但仍旧是不可忽视的，加上进程间数据同步远比不上线程间同步的高效，所以也不是一种完美的方案。不过 epoll则没有这个限制，它所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048,举个例子,在1GB内存的机器上大约是10万左右，具体数目可以cat /proc/sys/fs/file-max察看,一般来说这个数目和系统内存关系很大。
2. IO效率不随FD数目增加而线性下降
传统的select/poll另一个致命弱点就是当你拥有一个很大的socket集合，不过由于网络延时，任一时间只有部分的socket是"活跃"的，但是select/poll每次调用都会线性扫描全部的集合，导致效率呈现线性下降。但是epoll不存在这个问题，它只会对"活跃"的socket进行操作---这是因为在内核实现中epoll是根据每个fd上面的callback函数实现的。那么，只有"活跃"的socket才会主动的去调用 callback函数，其他idle状态socket则不会，在这点上，epoll实现了一个"伪"AIO，因为这时候推动力在os内核。在一些 benchmark中，如果所有的socket基本上都是活跃的---比如一个高速LAN环境，epoll并不比select/poll有什么效率，相反，如果过多使用epoll_ctl,效率相比还有稍微的下降。但是一旦使用idle connections模拟WAN环境,epoll的效率就远在select/poll之上了。
3. 使用mmap加速内核与用户空间的消息传递。
这点实际上涉及到epoll的具体实现了。无论是select,poll还是epoll都需要内核把FD消息通知给用户空间，如何避免不必要的内存拷贝就很重要，在这点上，epoll是通过内核于用户空间mmap同一块内存实现的。
4. 内核微调
这一点其实不算epoll的优点了，而是整个linux平台的优点。也许你可以怀疑linux平台，但是你无法回避linux平台赋予你微调内核的能力。比如，内核TCP/IP协议栈使用内存池管理sk_buff结构，那么可以在运行时期动态调整这个内存pool(skb_head_pool)的大小 --- 通过echo XXXX>/proc/sys/net/core/hot_list_length完成。再比如listen函数的第2个参数(TCP完成3次握手的数据包队列长度)，也可以根据你平台内存大小动态调整。更甚至在一个数据包面数目巨大但同时每个数据包本身大小却很小的特殊系统上尝试最新的NAPI网卡驱动架构。

epoll api函数比较简单，包括创建一个epoll描述符，添加监听事件，阻塞等待所监听的事件发生,关闭epoll描述符，如下：

``` c

#include <sys/epoll.h>
#include <unistd.h> 
int epoll_create(int size); //epoll描述符
int close(int fd);//关闭epoll描述符

```

创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大。需要注意的是，当创建好epoll句柄后，epoll本身就占用一个fd值，所以用完后必须调用close()关闭，以防止fd被耗尽。

```c
#include <sys/epoll.h>
#include <unistd.h> 
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);//添加监听事件 typedef union epoll_data {    
  void        *ptr;    
  int          fd;    
  uint32_t     u32;    
  uint64_t     u64;
} 
epoll_data_t; 
struct epoll_event {    
  uint32_t     events;      /* Epoll events */    
  epoll_data_t data;        /* User data variable */
};
```

epoll_ctl为事件注册函数，第一个参数是epoll_create()的返回值，第二个参数表示动作，用三个宏来表示：

EPOLL_CTL_ADD：注册新的fd到epfd中；
EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
EPOLL_CTL_DEL：从epfd中删除一个fd;
第三个参数是需要监听的描述符fd,第四个参数是告诉内核需要监听什么事件，struct epoll_event结构如上所示，其中events为需要注册的事件，可以为下面几个宏的集合：

EPOLLIN：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
EPOLLOUT：表示对应的文件描述符可以写；
EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
EPOLLERR：表示对应的文件描述符发生错误；
EPOLLHUP：表示对应的文件描述符被挂断；
EPOLLET：将EPOLL设为边缘触发（Edge Triggered）模式，这是相对于水平触发（Level Triggered）来说的。详细见下面描述
EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里。
操作系统对被注册的文件描述符的事件发送以后有两种处理方式，分别有LT和ET模式，默认情况下为LT，如果需要设置为ET模式，设置struct epoll_event.events|EPOLLET。

LT(level triggered)是缺省的工作方式，并且同时支持block和no-block socket.在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的，所以，这种模式编程出错误可能性要小一点，传统的select/poll都是这种模型的代表。
ET (edge-triggered)是高速工作方式，只支持no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了(比如，你在发送，接收或者接收请求，或者发送接收的数据少于一定量时导致了一个EWOULDBLOCK 错误）。但是请注意，如果一直不对这个fd作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知(only once),不过在TCP协议中，ET模式的加速效用仍需要更多的benchmark确认。
struct epoll_event.data为一个epoll_data_t类型的联合体，详细结构如上，其中可以保存指针，描述符，32/64位的整数。如果在监听过程中，该描述符上面有相应的事件发生，系统将会把该字段返回。先看监听函数吧。看完就知道

异步IO详解
---
