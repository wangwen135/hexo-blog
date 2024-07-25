---
title: I-O多路复用（I-O-Multiplexing）
date: 2021-08-16 23:41
tags: 
  - Netty
categories:
  - [Netty]
---



我们希望在一个或多个I/O条件就绪（即输入已准备好被读取，或者描述符能够接收更多输出）时得到通知。此功能称为I/O复用，由select和poll等函数支持。

通常情况下，I/O多路复用通常用于网络应用程序：  
- 当客户端处理多个描述符时（通常是交互式输入和网络套接字）
- 当客户端同时处理多个套接字时（这是可能的，但很少见）
- 如果TCP服务器同时处理侦听套接字及其连接的套接字
- 如果服务器同时处理TCP和UDP
- 如果服务器处理多种服务，并且可能处理多种协议

但是I/O复用不限于网络编程。

> Unix/Linux系统中的所有内容都是文件。每个进程都有一个文件描述符，该描述符指向文件，套接字，设备和其他操作系统对象。

> 我们告诉内核我们对哪些描述符感兴趣（用于读取，写入或异常情况）以及等待多长时间。我们感兴趣的描述符不限于套接字，还可以是任何的描述符。

## I/O模型

Unix/Linux 下可供我们使用的五个I/O模型
1. blocking I/O 【阻塞I/O】
2. nonblocking I/O 【非阻塞I/O】
3. I/O multiplexing (select and poll) 【多路复用I/O】
4. signal driven I/O (SIGIO) 【信号驱动】
5. asynchronous I/O (the POSIX aio_ functions) 【异步I/O，由POSIX规范定义的】


输入操作通常有两个不同的阶段：

1. 等待数据准备就绪。这涉及等待数据到达网络。数据包到达时，它将被复制到内核中的缓冲区中。
2. 将数据从内核复制到进程。这意味着将（就绪）数据从内核缓冲区复制到我们的应用程序缓冲区中


### 同步IO&异步IO
根据 POSIX 定义:
- 同步I/O操作会阻塞请求进程，直到I/O操作完成
- 异步I/O操作不会导致阻塞请求进程

按照这种分类，上边5种I/O模型中，只有AIO一种是异步的，其他都是同步的。  
因为其中真正的IO操作(recvfrom 调用) 会阻塞进程，recvfrom 会阻塞等待内核将数据从内核空间复制到应用进程空间, 当赋值完成后, recvfrom 才返回。

但是从我们编程的角度来看，「I/O多路复用」和「信号驱动I/O」都不会导致我们的进程完全被阻塞，因为在多线程下，阻塞一个线程并不会导致整个进程被阻塞。


## I/O多路复用实现方式

I/O多路复用(I/O multiplexing) 通过系统调用 select，poll，epoll 支持。

select/epoll的好处就在于单个进程就可以同时处理多个网络连接的IO。它的基本原理就是select，poll，epoll这些个函数会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。  

>本质是通过系统函数select、poll、epool（模块）来监听多个文件描述符（套接字描述符）。   
>无论epoll还是select都受限于系统中单个进程能够打开的文件句柄数。

select/poll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。


### 一、select
select是个系统调用，提供了一种用于实现同步多路复用I/O的机制
```
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```
对select()的调用将一直阻塞，直到给定的文件描述符准备执行I/O为止，或者直到经过可选的指定超时为止。

监视的文件描述符类型分为三种：
- 监视readfds集中列出的文件描述符，以查看是否有数据可读取。
- 监视writefds集中列出的文件描述符，以查看写操作是否将完成而不会阻塞。
- 监视exceptionfds集中的文件描述符，以查看是否发生了异常或带外数据是否可用（这些状态仅适用于套接字）。

执行select()成功返回后，将修改每个集合，以使其仅包含准备好由该集合描述的I/O类型的文件描述符。


#### 优缺点
##### 优点：
select的主要优点是它具有很高的可移植性-像OS一样的每个UNIX都具有它


##### 缺点：
- select()使用fd_set来表示文件描述符的集合，而fd_set其实就是一个固定长度的位向量(bit vector)，在Linux上，这个固定长度是FD_SETSIZE，其数值是1024。
故select()监听的文件描述符总数必须小于1024。  
- 我们需要遍历文件描述符以检查它是否存在于select返回的集合中  

##### 注意事项：
每次select之前要重置每个入参集合的值（返回时会被修改）。




### 二、poll
poll提供与相似的功能select。  
与select()不同，因为select()具有效率低下的三个基于位掩码的文件描述符集，poll()使用nfds pollfd结构的单个数组。原型更简单：
```
int poll (struct pollfd *fds, unsigned int nfds, int timeout);
```

pollfd结构的事件和返回事件具有不同的字段，因此我们不必每次都构建它
```
struct pollfd {
      int fd;
      short events; 
      short revents;
};
```

对于每个文件描述符，构建一个类型为pollfd的对象，并填充所需的事件。poll返回后，检查revents字段即可

就像我们对select所做的那样，我们需要检查每个pollfd对象以查看其文件描述符是否已准备就绪，但是我们不需要在每次迭代时都构建集合


#### 优缺点
##### 优点： 
poll()系统调用将输入（events字段）与输出（revents字段）分开，从而允许入参被重复使用而无需更改。


##### 缺点：
由于某些Unix系统不支持poll()，因此select()具有更高的可移植性。


### select 和 poll 的性能问题
使用select()或poll()监听大量的文件描述符时，往往会遭遇到性能问题。当用户每次调用select()或poll()时，内核会对传入的所有文件描述符都检查一遍，并记录其中有哪些文件描述符存在I/O就绪，这个操作的耗时将随着文件描述符数量的增加而线性增长。

另一个重要因素也会影响select()和poll()的性能，例如用户每次调用poll()时，都需要传递一个pollfd数组，而poll()会将这个数组从用户空间拷贝到内核空间，当内核对这个数组作了修改之后，poll()又会将这个数组从内核空间拷贝到用户空间。随着pollfd数组长度的增加，每次拷贝的时间也会线性增长，一旦poll()需要监听大量的文件描述符，每次调用poll()时，这个拷贝操作将带来不小的开销。这个问题的根源在于select()和poll()的API设计不当，例如，对于应用程序来说，它每次调用poll()所监听的文件描述符集合应该是差不多的，所以我们不禁这样想，如果内核愿意提供一个数据结构，记录程序所要监听的文件描述符集合，这样每次用户调用poll()时，poll()就不需要将pollfd数组拷贝来拷贝去了（没错，epoll 就是这样解决的）。


### 三、epoll
> epoll是为了解决select()和poll()中存在的问题

epoll是个模块，由三个系统调用（epoll_create epoll_ctl epoll_wait）组成，Epoll 系统调用可帮助我们在内核中创建和管理上下文。epoll使用红黑树（RB-tree）数据结构来跟踪当前正在监视的所有文件描述符。

我们将任务分为3个步骤：
- 使用epoll_create在内核中创建上下文
- 使用epoll_ctl向/从上下文中添加和删除文件描述符
- 使用epoll_wait等待上下文中的事件



epoll_ctl()负责增加、删除或修改红黑树上的节点，而epoll_wait()则负责返回双向链表中就绪的文件描述符(及其事件)。

当网卡收到一个 packet 的时候，会触发一个硬件中断，这导致内核调用相应的中断 handler，从网卡中读入数据放到协议栈，当数据量满足一定条件时，内核将回调ep_poll_callback()这个方法，它负责把这个就绪的文件描述符添加到双向链表中。这样当用户调用epoll_wait()时，epoll_wait()所做的就只是检查双向链表是否为空，如果不为空，就把文件描述符和数量返回给用户即可。


#### 触发模式
epoll提供边沿触发（Edge-Triggered）及状态触发（Level-Triggered）模式。

> Level-Triggered 也翻译成水平触发、条件触发

**边沿触发：**  
监控对象的状态发生改变时触发，此后如果状态一直没有发生变化应用程序将不再收到通知

**状态触发：**  
处于某种状态下（如缓冲区可以读）就一直触发


**如：**  
socket接收到缓存数据时，调用epoll_wait，上面两种方法都将返回，表明存在要读取的数据。  
假设读取器仅消耗了缓冲区中的部分数据。  
在状态触发模式下，epoll_wait只要管道的缓冲区包含要读取的数据，对epoll_wait的调用将立即返回；  
但是，在边沿触发模式下，epoll_wait仅在将新数据写入缓存区后才返回。



#### 一般编程逻辑
1. 【epoll_create】在内核中创建epoll实例并返回一个epoll文件描述符（epfd）。  
```
int epoll_create1(int flags);
```

2. 【epoll_ctl】向epfd对应的内核epoll实例添加、修改或删除对fd（File descriptor）上事件event的监听。类型可以为EPOLL_CTL_ADD, EPOLL_CTL_MOD, EPOLL_CTL_DEL分别对应的是添加新的事件，修改文件描述符上监听的事件类型。
```
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

3. 【epoll_wait】循环执行epoll_wait，当timeout 为0 时，epoll_wait 永远会立即返回。而timeout 为-1 时，epoll_wait 会一直阻塞直到任一已注册的事件变为就绪。当timeout 为一正整数时，epoll 会阻塞直到计时timeout 毫秒终了或已注册的事件变为就绪。因为内核调度延迟，阻塞的时间可能会略微超过timeout 毫秒。
```
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

#### 优缺点

##### 优点：
- 我们可以在等待时添加和删除文件描述符
- epoll_wait仅返回具有就绪文件描述符的对象
- epoll具有更好的性能-（Epoll时间复杂度为O(1) 代替之前的O(n)）
- epoll可以表现为状态触发或边缘触发
 

##### 缺点： 
epoll是特定于Linux的，因此不可移植




