---
title: Netty-笔记
date: 2023-04-22 23:41
tags: 
  - Netty
categories:
  - [Netty]
---

**基于netty 4.x**

**相关页面地址**：  
4.x用户指南：https://netty.io/wiki/user-guide-for-4.x.html  
Netty Example：https://github.com/netty/netty/tree/4.1/example/src/main/java/io/netty/example  
API：https://netty.io/4.1/xref/index.html

## Netty框架
Netty是一个基于NIO的异步事件驱动的网络应用程序框架和工具，使用他可以快速轻松的开发
出高性能和高扩展性的网络应用程序，例如自定义协议的服务器和客户端。它极大地简化和简化了网络编程，如TCP和UDP套接字服务器的开发。



## 线程模型与异步处理

### 线程模型
Reactor（反应堆模型）  
Reactor模式也叫Dispatcher模式，即I/O多路复用统一监听事件，收到事件后分发(Dispatch给某进程)

> Reactor就是一个执行while (true) { selector.select(); ...}循环的线程，会源源不断的产生新的事件

Reactor模型中有2个关键组成：
- Reactor Reactor在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理程序来对IO事件做出反应。
- Handlers 处理程序执行I/O事件要完成的实际事件


取决于Reactor的数量和Hanndler线程数量的不同，Reactor模型有3个变种
- 单Reactor单线程
- 单Reactor多线程
- 主从Reactor多线程

#### Netty线程模型
Netty主要基于主从Reactors多线程模型，其中Reactor分为：MainReactor和SubReactor：

- MainReactor负责客户端的连接请求，并将请求转交给SubReactor
- SubReactor负责相应通道的IO读写请求
- 非IO请求（具体逻辑处理）的任务则会直接写入队列，等待worker threads进行处理


虽然Netty的线程模型基于主从Reactor多线程，但是实际实现上SubReactor和Worker线程在同一个线程池中

- bossGroup线程池则只是在bind某个端口后，获得其中一个线程作为MainReactor，专门处理端口的accept事件，每个端口对应一个boss线程
- workerGroup线程池会被各个SubReactor和worker线程充分利用


### 异步处理
Netty中的I/O操作是异步的，包括bind、write、connect等操作会简单的返回一个ChannelFuture，调用者并不能立刻获得结果，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。

- 通过isDone方法来判断当前操作是否完成
- 通过isSuccess方法来判断已完成的当前操作是否成功
- 通过getCause方法来获取已完成的当前操作失败的原因
- 通过isCancelled方法来判断已完成的当前操作是否被取消
- 通过addListener方法来注册监听器，当操作已完成(isDone方法返回完成)，将会通知指定的监听器；如果future对象已完成，则立即通知指定的监听器

---


## 内存

Netty中使用了自己实现的ByteBuffer，直接内存，内存池化，引用计数器等技术，重点要看一下 io.netty.buffer.ByteBuf



### ByteBuffer

零个或多个字节（八位字节）的随机且顺序可访问的序列。此接口提供一个或多个原始字节数组（byte []）和NIO buffers 的抽象视图。



#### 创建缓冲区
建议在Unpooled中使用辅助方法创建一个新的缓冲区，而不是调用单个实现的构造函数。



#### 随机存取索引
就像普通的原始字节数组一样，ByteBuf使用从零开始的索引。这意味着第一个字节的索引始终为0，而最后一个字节的索引始终为-1。例如，要迭代缓冲区的所有字节，您可以不管其内部实现如何，请执行以下操作：

```
 ByteBuf buffer = ...;
 for (int i = 0; i < buffer.capacity(); i ++) {
     byte b = buffer.getByte(i);
     System.out.println((char) b);
 }
```



#### 顺序访问索引
ByteBuf提供了两个指针变量来支持顺序读取和写入操作；分别是针对读操作的readerIndex和针对写操作的writerIndex。    

> 无需像使用JDK中的Buffer那样手动flip()



**读操作：**  

名称以read或skip开头的任何操作都将获取或跳过当前readerIndex处的数据，并将其增加读取字节数。如果读取操作的参数是ByteBuf，并且未指定目标索引，则指定缓冲区的writerIndex会一起增加。  
如果没有足够的内容，则会引发IndexOutOfBoundsException。  
新分配，包装或复制的缓冲区的readerIndex的默认值为0。
```
// 迭代缓冲区的可读字节
ByteBuf buffer = ...;
while (buffer.isReadable()) {
    System.out.println(buffer.readByte());
}
```


**写操作：**

名称以write开头的任何操作都将在当前writerIndex处写入数据，并将增加其writerIndex的计数。如果写操作的参数是ByteBuf，并且未指定源索引，则指定缓冲区的readerIndex会一起增加。  
如果没有足够的可写字节，则引发IndexOutOfBoundsException。新分配的缓冲区的writerIndex的默认值为0。包装或复制的缓冲区的writerIndex的默认值为缓冲区的容量。

```
// 用随机整数填充缓冲区的可写字节
ByteBuf buffer = ...;
while (buffer.maxWritableBytes() >= 4) {
    buffer.writeInt(random.nextInt());
}
```



**丢弃已经读取的字节：**

通过调用discardReadBytes()来丢弃已经读取的字节，以回收使用的字节区域。

示例：

```
  调用 discardReadBytes() 之前 （已经读取了一定的字节）

      +-------------------+------------------+------------------+
      | discardable bytes |  readable bytes  |  writable bytes  |
      +-------------------+------------------+------------------+
      |                   |                  |                  |
      0      <=      readerIndex   <=   writerIndex    <=    capacity


  调用 discardReadBytes()之后 （相关指针前移，已经读取过的内容被丢弃）

      +------------------+--------------------------------------+
      |  readable bytes  |    writable bytes (got more space)   |
      +------------------+--------------------------------------+
      |                  |                                      |
 readerIndex (0) <= writerIndex (decreased)        <=        capacity
```



**清除缓冲区索引：**

可以通过调用clear()将readerIndex和writerIndex都设置为0，它不会清除缓冲区内容（例如填充0），而只是清除两个指针。

> **请注意，此操作的语义与java.nio.ByteBuffer.clear()不同。**



#### 搜索操作

对于简单的单字节搜索，请使用indexOf(int，int，byte) 和 bytesBefore(int，int，byte) 。当处理以NUL结尾的字符串时，bytesBefore(byte) 特别有用。对于复杂的搜索，请使用带有ByteProcessorimplementation的forEachByte(int，int，ByteProcessor)。



#### 标记和重置
每个缓冲区中都有两个标记索引。一种用于存储readerIndex，另一种用于存储writerIndex。您始终可以通过调用reset方法来重新定位两个索引之一。除了没有readlimit以外，它的工作方式与InputStream中的mark和reset方法类似。



#### 派生缓冲区

您可以通过调用以下方法之一来创建现有缓冲区的视图：

- duplicate()
- slice()
- slice(int, int)
- readSlice(int)
- retainedDuplicate()
- retainedSlice()
- retainedSlice(int, int)
- readRetainedSlice(int)

派生的缓冲区将具有独立的readerIndex，writerIndex和标记索引，而它共享其他内部数据表示形式，就像NIO缓冲区一样。  
如果需要现有缓冲区的全新副本，请调用 copy( ) 方法。



#### 非保留和保留派生缓冲区

请注意，plicate()，slice()，slice(int，int)和readSlice(int) 不会在返回的派生缓冲区上调用retain()，因此不会增加其引用计数。如果需要创建具有增加的引用计数的派生缓冲区，请考虑使用retainedDuplicate()，retainedSlice()，retainedSlice(int，int) 和 readRetainedSlice(int)，这可能会返回产生较少垃圾的缓冲区实现。



#### 转换为现有的JDK类型

**字节数组**

如果ByteBuf由字节数组（即byte []）支持，则可以直接通过array()方法访问它。要确定缓冲区是否由字节数组支持，应使用hasArray()。

**NIO缓冲区**

如果ByteBuf可以转换为共享其内容的NIO ByteBuffer（即视图缓冲区），则可以通过nioBuffer()方法获取它。要确定是否可以将缓冲区转换为NIO缓冲区，请使用nioBufferCount()。

**String**

各种toString(Charset)方法将ByteBufin转换为String。请注意，toString()不是转换方法。

**I/O流**

请参考ByteBufInputStream和ByteBufOutputStream。



----



### 计数引用

ByteBuf 实现了 ReferenceCounted 接口

在Netty handler的设计规范中，所有输入的数据在处理结束时都会调用ReferenceCountUtil.release()释放，只是具体释放方法根据池化、非池化、直接内存、堆内存的不同实现不同。

#### ReferenceCounted

一个引用计数的对象，需要显式取消分配。

实例化一个新的ReferenceCounted时，它以引用计数为1开头。keep() 增加引用计数，release() 减少引用计数。  
如果引用计数减小为0，则将显式释放该对象，并且 访问已释放对象通常会导致访问冲突。

如果实现ReferenceCounted的对象是其他实现ReferenceCounted的对象的容器，则当容器的引用计数变为0时，包含的对象也将通过release() 释放。



#### ReferenceCountUtil

一个工具方法集合，处理的对象需要实现 ReferenceCounted




### 直接内存（堆外内存）



实现零拷贝，既使用时无需将内存从系统空间复制到用户空间

主要是指接收和发送 ByteBuffer 使用虚拟机的堆外内存，不用将内存内容拷贝到jvm的堆中





### 内存块池化
对于堆外内存的申请分配以及释放是一个比较耗时的操作，不能像jvm中一样快（jvm已经提前像系统申请了一块内存空间）  
Netty为了重用这些内存，将内存分成一块一块的小内存，然后将内存块像线程池中的线程一样重复使用，来避免重复的分配和回收操作。（据说性能超高，没有测试过）  




### ByteBufAllocator

实现负责分配缓冲区。这个接口的实现应该是线程安全的。

#### UnpooledByteBufAllocator
非池化内存分配器

#### PooledByteBufAllocator
池化内存分配器



###  io.netty.buffer.Unpooled

通过分配新空间或包装或复制现有字节数组、字节缓冲区和字符串来创建新的ByteBuf。

**静态方法**

```
import static io.netty.buffer.Unpooled.*;

ByteBuf heapBuffer    = buffer(128);
ByteBuf directBuffer  = directBuffer(256);
ByteBuf wrappedBuffer = wrappedBuffer(new byte[128], new byte[256]);
ByteBuf copiedBuffer  = copiedBuffer(ByteBuffer.allocate(128));
```

**分配新缓冲区**  
buffer(int) 分配一个新的固定容量堆缓冲区。  
directBuffer(int) 分配一个新的固定容量直接缓冲区。


**创建包装的缓冲区**  
包装缓冲区是一种缓冲区，它是一个或多个现有字节数组和字节缓冲区的视图。原始数组或缓冲区内容的任何更改都将在包装的缓冲区中可见。提供了各种包装方法，它们的名称全为wrappedBuffer()。如果要创建一个由多个数组组成的缓冲区以减少内存副本的数量，则可能需要仔细研究一下接受varargs的方法。

**创建复制的缓冲区**  
复制缓冲区是一个或多个现有字节数组，字节缓冲区或字符串的深层副本。与包装的缓冲区不同，原始数据和复制的缓冲区之间没有共享数据。提供了各种复制方法，它们的名称全为copyedBuffer()。使用此操作将多个缓冲区合并为一个缓冲区也很方便。





---

## 重点类介绍

### 启动辅助类

#### ServerBootstrap 
**服务端用** 

ServerBootstrapAcceptor的channelRead方法，
会将进入的连接注册到workerGroup （NioEventLoopGroup）中，之后的读写以及业务操作就是NioEventLoopGroup中的EventLoop来处理了

注册时会有一定的机制（DefaultEventExecutorChooserFactory，就是轮询）来决定用哪一个EventLoop来处理本次连接


#### Bootstrap 
**客户端用** 

bind()方法与无连接传输如数据报（UDP）结合使用非常有用。对于常规TCP连接，请使用提供的connect()方法。


### Channel （通道）
网络套接字或能够进行I/O操作(如读、写、连接和绑定)的组件的连接。


**通道为用户提供:**  
- 通道的当前状态(例如：是否打开，是否是连接状态)
- 通道的配置参数(例如接收缓冲区大小)
- 通道支持的I/O操作(例如，读、写、连接和绑定)
- ChannelPipeline处理与通道相关的所有I/O事件和请求。

**所有I/O操作都是异步的**  
Netty中的所有I/O操作都是异步的。这意味着任何I/O调用将立即返回，而不能保证所请求的I/O操作在调用结束时已经完成。返回一个ChannelFuture实例，该实例将在请求的I/O操作成功、失败或取消时通知你。


**释放资源**  
一旦Channel使用完毕，调用ChannelOutboundInvoker.close()或ChannelOutboundInvoker.close(ChannelPromise)释放所有资源非常重要。这样可确保以适当的方式释放所有资源，如文件描述符。

#### SocketChannel
TCP/IP套接字通道

两个常用子类  
**NioServerSocketChannel**  用于服务端，它使用基于NIO选择器的实现来接受新连接。  
**NioSocketChannel** 用于客户端，它使用基于NIO选择器的实现  


#### DatagramChannel
UDP/IP 通道

常用子类
**NioDatagramChannel** NIO的UDP数据包通道



### NioEventLoop （事件循环）
SingleThreadEventLoop的实现，它将Channel注册到选择器中，并且在事件循环中对它们进行多路复用。  
该类在单个线程中以有序/串行的方式执行提交的所有任务。


本质上是一个单线程的线程池，通过execute方法提交的任务都将被这个Thread线程来执行

在注册时会执行execute方法，调用doStartThread，启动事件循环

>  调用doStartThread 只会执行一次

代码在：
> SingleThreadEventExecutor.doStartThread()  
> SingleThreadEventExecutor.this.run();


里面有个固定无限循环，来判断selector中是否有事件

就像我们写NOI的那个循环一样  
> NioEventLoop.run()

> 还实现了：ScheduledExecutorService 调度线程池

channel 会绑定到一个eventLoop中，这个channel对应的所有操作都会由这个eventLoop来执行,
包括所有的IO操作和handler中业务逻辑

但是多个channel 可能会被分配到同一个eventLoop中

因为是单线程的，它将以有序/串行方式处理所有提交的任务，这样不必要考虑并发同步的问题。



### NioEventLoopGroup （事件循环组）
MultithreadEventLoopGroup的实现，用于基于NIO 选择器的Channel。  
它同时使用多个线程处理它的任务，通过它的next()方法提供要使用的EventExecutor。除此之外，它还负责处理它们的生命周期，并允许以全局方式关闭它们。

> new NioEventLoopGroup() 没有参数时:
> - 默认的NioEventLoop的数量是机器CPU数量的两倍
> - 默认的拒绝策略是：拒绝（抛异常）
> - eventLoop做事件循环时的Selector如果没有指定将使用系统默认的，如windows的WindowsSelectorProvider


NioEventLoopGroup是用来管理NioEventLoop的，里面有一个 <code>EventExecutor[] children; </code>管理着这个组下的全部事件循环EventLoop对象


初始化时会实例化全部的NioEventLoop对象，通过 <code>EventLoop newChild(Executor executor, Object... args)</code> 方法


channel初始化时会进行注册，注册通道时调用 <code>next().register(channel)</code>


服务端程序在 ServerBootstrapAcceptor 的channelRead 方法中将新的channel注册到workerGroup中


next方法会通过EventExecutorChooser（默认轮询）来获取EventExecutor（NioEventLoop）



> **bossGroup** 只需要一个 NioEventLoop 就可以了，只负责将channel注册到 workerGroup 中  
> **workerGroup** 默认CPU核心数*2 个NioEventLoop


### ChannelPipeline （管道）
ChannelHandler的列表，用于处理或拦截Channel的入站事件和出站操作。
ChannelPipeline实现了一种高级的拦截、过滤模式，以使用户可以完全控制事件的处理方式以及管道中的ChannelHandler如何相互交互。

每个Channel都有其自己的ChannelPipeline，并且在创建新Channel时会自动创建它。

下图描述了ChannelPipeline典型地ChannelHandler如何处理I/O事件。 I/O事件由ChannelInboundHandler或ChannelOutboundHandler处理，并通过调用ChannelHandlerContext中定义的事件传播方法（例如ChannelHandlerContext.fireChannelRead(Object)和ChannelHandlerContext.write(Object)）转发到其最近的处理程序。

```
                                                 I/O Request
                                            via {@link Channel} or
                                        {@link ChannelHandlerContext}
                                                      |
  +---------------------------------------------------+---------------+
  |                           ChannelPipeline         |               |
  |                                                  \|/              |
  |    +---------------------+            +-----------+----------+    |
  |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  |               |
  |               |                                  \|/              |
  |    +----------+----------+            +-----------+----------+    |
  |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  .               |
  |               .                                   .               |
  | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
  |        [ method call]                       [method call]         |
  |               .                                   .               |
  |               .                                  \|/              |
  |    +----------+----------+            +-----------+----------+    |
  |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  |               |
  |               |                                  \|/              |
  |    +----------+----------+            +-----------+----------+    |
  |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  |               |
  +---------------+-----------------------------------+---------------+
                  |                                  \|/
  +---------------+-----------------------------------+---------------+
  |               |                                   |               |
  |       [ Socket.read() ]                    [ Socket.write() ]     |
  |                                                                   |
  |  Netty Internal I/O Threads (Transport Implementation)            |
  +-------------------------------------------------------------------+
```


入站事件由入站处理程序在自下而上的方向上进行处理，如该图的左侧所示。入站处理程序通常处理由图底部的I/O线程生成的入站数据。通常通过实际的输入操作（例如SocketChannel.read（ByteBuffer））从远程对等方读取入站数据。如果入站事件超出了顶部入站处理程序的范围，则将其静默丢弃，或者在需要引起注意时将其记录下来。

出站事件由出站处理程序按自上而下的方向进行处理，如该图的右侧所示。出站处理程序通常会生成或转换出站流量（例如写请求）。如果出站事件超出了底部出站处理程序，则由与通道关联的I/O线程处理。 I/O线程通常执行实际的输出操作，例如SocketChannel.write（ByteBuffer）。


假设我们创建了如下的管道
```
ChannelPipeline p = ...;
p.addLast("1", new InboundHandlerA());
p.addLast("2", new InboundHandlerB());
p.addLast("3", new OutboundHandlerA());
p.addLast("4", new OutboundHandlerB());
p.addLast("5", new InboundOutboundHandlerX());
```

在上面的示例中，名称以Inbound开头的类表示它是一个入站处理程序。名称以Outbound开头的类表示它是一个出站处理程序。  
在给定的示例配置中，事件进入时处理程序的评估顺序为1、2、3、4、5。事件离开时，处理程序的评估顺序为5、4、3、2、1。   

ChannelPipeline跳过某些处理程序的评估，以缩短堆栈深度：
- 3和4没有实现ChannelInboundHandler，因此入站事件的实际评估顺序为：1、2和5。
- 1和2没有实现ChannelOutboundHandler，因此出站事件的实际评估顺序为：5、4和3。
- 如5同时实现ChannelInboundHandler和ChannelOutboundHandler，则入站和出站事件的评估顺序可能分别为1 2 5和5 4 3。


ChannelPipeline是线程安全的，也就是说，我们可以动态的添加、删除其中的ChannelHandler。考虑这样的场景：服务器需要对用户登录信息进行加密，而其他信息不加密，则可以首先将加密Handler添加到ChannelPipeline，验证完用户信息后，主动从ChnanelPipeline中删除，从而实现该需求



### ChannelHandlerContext （处理器的上下文）
channelHandler 的上下文对象，使ChannelHandler可以与ChannelPipeline中的其他Handler进行交互。  
使用上下文对象，ChannelHandler可以在上下游传递事件，动态修改管道（在管道中动态的插入或删除ChannelHandler）或存储特定于处理程序的信息（使用AttributeKeys）。


**结合ChannelPipeline**  
处理程序必须调用ChannelHandlerContext中的事件传播方法，以将事件转发到其下一个处理程序。这些方法包括：

入站事件传播方法：
```
ChannelHandlerContext.fireChannelRegistered()
ChannelHandlerContext.fireChannelActive()
ChannelHandlerContext.fireChannelRead(Object)
ChannelHandlerContext.fireChannelReadComplete()
ChannelHandlerContext.fireExceptionCaught(Throwable)
ChannelHandlerContext.fireUserEventTriggered(Object)
ChannelHandlerContext.fireChannelWritabilityChanged()
ChannelHandlerContext.fireChannelInactive()
ChannelHandlerContext.fireChannelUnregistered()
```
出站事件传播方法：
```
ChannelHandlerContext.bind(SocketAddress，ChannelPromise)
ChannelHandlerContext.connect(SocketAddress，SocketAddress，ChannelPromise)
ChannelHandlerContext.write(Object，ChannelPromise)
ChannelHandlerContext.flush()
ChannelHandlerContext.read()
ChannelHandlerContext.disconnect(ChannelPromise)
ChannelHandlerContext.close(ChannelPromise)
ChannelHandlerContext.deregister(ChannelPromise)
```

如何进行事件传播:
```
public class MyInboundHandler extends ChannelInboundHandlerAdapter {
     @Override
     public void channelActive(ChannelHandlerContext ctx) {
         System.out.println("Connected!");
         ctx.fireChannelActive();
     }
 }

 public class MyOutboundHandler extends ChannelOutboundHandlerAdapter {
     @Override
     public void close(ChannelHandlerContext ctx, ChannelPromise promise) {
         System.out.println("Closing ..");
         ctx.close(promise);
     }
 }
```



### ChannelInitializer（通道初始化器）
一个特殊的ChannelHandler，用于给ChannelPipeline绑定handler  
在连接建立时会调用handlerAdded方法然后调用initChannel方法，然后执行我们编写的  
channelPipeline.addLast(new StringDecoder())  
等代码，这里使用的是new 一个新对象，这样就能给每一个连接创建独立的handler实例了，这些实例就可以保存各种状态



---




### ChannelHandler（处理器）

处理网络IO事件，对消息进行编码解码，和一定的业务逻辑处理。我们主要就实现各种Handler。  

>过长时间的业务逻辑代码如果编写在ChannelHandler中，不能直接在workerGroup线程池中处理，可能阻塞其他的IO操作
>通过外部线程池，或指定Handler的执行线程池来处理

通常必须实现下面的两个子类之一：

1. ChannelInboundHandler 处理入站I/O事件
2. ChannelOutboundHandler 处理出站I/O事件

另外还提供了以下适配器类：  
- ChannelInboundHandlerAdapter 处理入站I/O事件
- ChannelOutboundHandlerAdapter 处理出站I/事件
- ChannelDuplexHandler 可以处理入站和出站事件


#### Handler中的状态管理
##### 1. 在Handler实现类中使用成员变量控制状态
```
public class DataServerHandler extends SimpleChannelInboundHandler<Message> {

     private boolean loggedIn;

     @Override
     public void channelRead0(ChannelHandlerContext ctx, Message message) {
         if (message instanceof LoginMessage) {
             authenticate((LoginMessage) message);
             loggedIn = true;
         } else (message instanceof GetDataMessage) {
             if (loggedIn) {
                 ctx.writeAndFlush(fetchSecret((GetDataMessage) message));
             } else {
                 fail();
             }
         }
     }
     ...
 }

```
因为处理程序实例具有专用于一个连接的状态变量，所以您必须为每个新通道创建一个新的处理程序实例
```
 public class DataServerInitializer extends ChannelInitializer<Channel> {
     @Override
     public void initChannel(Channel channel) {
         channel.pipeline().addLast("handler", new DataServerHandler());
     }
 }
```
##### 2. 使用ChannelHandlerContext提供的AttributeKey
当不想创建许多处理器的实例时，用可以使用ChannelHandlerContext提供的AttributeKey来存储程序的状态信息

```
@Sharable
 public class DataServerHandler extends SimpleChannelInboundHandler<Message> {
     private final AttributeKey<Boolean> auth =
           AttributeKey.valueOf("auth");

     @Override
     public void channelRead(ChannelHandlerContext ctx, Message message) {
         Attribute<Boolean> attr = ctx.attr(auth);
         if (message instanceof LoginMessage) {
             authenticate((LoginMessage) o);
             attr.set(true);
         } else (message instanceof GetDataMessage) {
             if (Boolean.TRUE.equals(attr.get())) {
                 ctx.writeAndFlush(fetchSecret((GetDataMessage) o));
             } else {
                 fail();
             }
         }
     }
     ...
 }
```
只需要一个处理器的实例即可
```
 public class DataServerInitializer extends ChannelInitializer<Channel> {

     private static final DataServerHandler SHARED = new DataServerHandler();

     @Override
     public void initChannel(Channel channel) {
         channel.pipeline().addLast("handler", SHARED);
     }
 }

```
要注意这些ChannelHandler的线程安全性

#### @Sharable注解
**提供此注释是为了文档目的，就像JCIP注释一样。实际上是否创建多个实例取决于上面的两种写法**

如果ChannelHandler使用@Sharable注解进行标注，则表示可以只创建一次该处理程器的实例，然后将其多次添加到一个或多个ChannelPipelines中

如果未指定此注释，则每次将其添加到管道时都必须创建一个新的处理程序实例，因为它具有未共享的状态，例如成员变量。

---




## 其他

### ChannelOption

ChannelOption允许以类型安全的方式配置ChannelConfig。 支持哪个ChannelOption取决于ChannelConfig的实际实现，并且可能取决于其所属传输的性质。

### ChannelConfig

通道的一组配置属性

请向下转换为更特定的配置类型，例如SocketChannelConfig或使用setOptions（Map）设置特定于传输的属性：

```
 Channel ch = ...;
 SocketChannelConfig cfg = (SocketChannelConfig) ch.getConfig();
 cfg.setTcpNoDelay(false);
```



**Option map**

选项map 属性是动态只写属性，它允许配置Channel而不向下转换其关联的ChannelConfig。要更新选项map，请调用setOptions(Map)。
所有ChannelConfig都具有以下选项：

| 名称  | 关联的setter方法  |
| ---- | ---- |
| ChannelOption.CONNECT_TIMEOUT_MILLIS | setConnectTimeoutMillis(int)  |
| ChannelOption.WRITE_SPIN_COUNT | setWriteSpinCount(int)  |
| ChannelOption.WRITE_BUFFER_WATER_MARK | setWriteBufferWaterMark(WriteBufferWaterMark)  |
| ChannelOption.ALLOCATOR | setAllocator(ByteBufAllocator) |
| ChannelOption.AUTO_READ | setAutoRead(boolean)  |



#### SocketChannelConfig

**可用选项**

除了ChannelConfig提供的选项外，SocketChannelConfig允许在选项映射中使用以下选项:

| 名称                             | 关联的setter方法             |
| -------------------------------- | ---------------------------- |
| ChannelOption.SO_KEEPALIVE       | setKeepAlive(boolean)        |
| ChannelOption.SO_REUSEADDR       | setReuseAddress(boolean)     |
| ChannelOption.SO_LINGER          | setSoLinger(int)             |
| ChannelOption.TCP_NODELAY        | setTcpNoDelay(boolean)       |
| ChannelOption.SO_RCVBUF          | setReceiveBufferSize(int)    |
| ChannelOption.SO_SNDBUF          | setSendBufferSize(int)       |
| ChannelOption.IP_TOS             | setTrafficClass(int)         |
| ChannelOption.ALLOW_HALF_CLOSURE | setAllowHalfClosure(boolean) |



### 主动释放对象
这里主要指的是ByteBuf  

ByteBuf是一个引用计数对象（ReferenceCounted），必须通过该release()方法显式释放它。请记住，释放任何传递给处理程序的引用计数对象是处理程序的责任。通常，channelRead()处理程序方法的实现如下：
```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    try {
        // Do something with msg
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```




### 读数据

#### 从字节流到数据包（碎片化问题）
在基于流的传输（例如TCP / IP）中，将接收到的数据存储到套接字接收缓冲区中。基于流的传输的缓冲区不是数据包队列而是字节队列。这意味着，即使您将两个消息作为两个独立的数据包发送，操作系统也不会将它们视为两个消息，而只是一堆字节。因此，不能保证你读到的内容与远程对等方写的完全一样。

例如，让我们假设操作系统的TCP/IP堆栈已收到三个数据包：  
>【ABC】【DEF】【GHI】  

由于是基于流的传输协议，因此很有可能在你的应用程序中以以下分段形式读取它们：  
>【AB】【CDEFG】【H】【I】  

因此，无论是服务器端还是客户端，接收方都应将接收到的数据整理到一个或多个有意义的帧中，以使应用程序逻辑易于理解。在上面的示例中，接收到的数据应采用以下格式：  
>【ABC】【DEF】【GHI】 

##### 处理方案一
创建一个内部累积缓冲区，然后等待直到所有的字节都被接收到内部缓冲区中为止
```
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    private ByteBuf buf;
    
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        buf = ctx.alloc().buffer(4); // (1)
    }
    
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) {
        buf.release(); // (1)
        buf = null;
    }
    
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg;
        buf.writeBytes(m); // (2)
        m.release();
        
        if (buf.readableBytes() >= 4) { // (3)
            long currentTimeMillis = (buf.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        }
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```
1. 一个ChannelHandler有两个生命周期侦听器方法：handlerAdded()和handlerRemoved()。可以执行任意（取消）初始化任务，只要它不会长时间阻塞即可。
2. 应将所有接收到的数据累加到中buf
3. 然后，处理程序必须检查是否buf有足够的数据（在此示例中为4个字节），然后继续进行实际的业务逻辑。否则，Netty将channelRead()在有更多数据到达时再次调用该方法，最终将累加所有4个字节。

##### 处理方案二
添加多个ChannelHandler到ChannelPipeline，拆分一个ChannelHandler成多个，使其模块化减少应用程序的复杂性。

**使用编码解码器**
```
package io.netty.example.time;

public class TimeDecoder extends ByteToMessageDecoder { // (1)
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) { // (2)
        if (in.readableBytes() < 4) {
            return; // (3)
        }
        
        out.add(in.readBytes(4)); // (4)
    }
}
```
1. ByteToMessageDecoder是一个实现ChannelInboundHandler，可以轻松处理碎片问题。
2. ByteToMessageDecoderdecode()每当接收到新数据时，都使用内部维护的累积缓冲区调用该方法。
3. decode()可以决定out不向累积缓冲区中没有足够数据的位置添加任何内容。收到更多数据时ByteToMessageDecoder将decode()再次调用。
4. 如果decode()将对象添加到out，则表示解码器成功解码了一条消息。ByteToMessageDecoder将丢弃累积缓冲区的读取部分。请记住，您不需要解码多条消息。ByteToMessageDecoder会一直调用该decode()方法，直到该方法不添加任何内容out。

**累积缓冲区有两种实现**
1. MERGE_CUMULATOR  
通过使用内存副本，将它们合并到一个ByteBuf中来累积ByteBufs。
2. COMPOSITE_CUMULATOR  
通过将bytebuf添加到CompositeByteBuf中来累积bytebuf，因此尽可能不进行内存复制。注意CompositeByteBuf使用了一个更复杂的索引实现，所以取决于你的用例和解码器实现，这可能比仅仅使用merge_accumulator要慢。


**或者使用重复解码器 ReplayingDecoder**

ByteToMessageDecoder的变体，使得能够在阻塞I/O范例中实现非阻塞解码器。  
ReplayingDecoder您可以像已收到所有必需字节一样实现decode()和decodeLast()方法，而不用检查所需字节的可用性。
```
public class TimeDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(
            ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        out.add(in.readBytes(4));
    }
}
```
ReplayingDecoder通过一个特殊的ByteBuf 实现，Error当缓冲区中没有足够的数据时，该实现将抛出某种类型的实现。在上面的例子中只是假设调用时缓冲区中将有4个或更多字节。如果缓冲区中确实有4个字节，它将按预期返回整数标头。否则将引发Error。如果ReplayingDecoder捕获到 Error，则它将readerIndex把缓冲区的内容倒回到“初始”位置（即缓冲区的开头），并decode(..)在缓冲区中接收到更多数据时再次调用该方法。

详见：https://netty.io/4.1/api/io/netty/handler/codec/ReplayingDecoder.html

此外，Netty提供了开箱即用的解码器，使您能够非常轻松地实现大多数协议  
- io.netty.example.factorial  对于二进制协议
- io.netty.example.telnet  用于基于文本行的协议


#### 用POJO代替ByteBuf
在handler中将字节流转成对象，然后在调用链中传递，会使我们开发程序变得更方便

经过解码器，将字节流转成对象之后再out出去，在调用链后面的Handler中就可以直接取到对象了

如：   
解码器
```
@Override
protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    if (in.readableBytes() < 4) {
        return;
    }

    out.add(new UnixTime(in.readUnsignedInt()));
}
```
handler
```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    UnixTime m = (UnixTime) msg;
    System.out.println(m);
    ctx.close();
}
```


### 写数据
通过ChannelHandlerContext的write方法异步发送应答消息个客户端  
数据不直接写入SocketChannel，而是写入待发送数据的缓存区中 

ChannelHandlerContext的flush方法将队列中的消息写入到SockChannel中


### 优雅关闭
原理同线程池的关闭

channel.closeFuture().sync();

关闭所有事件循环以终止所有线程
```
bossGroup.shutdownGracefully();
workerGroup.shutdownGracefully();
```

### 耗时任务处理

bossGroup用于来接受连接，workGroup用于处理业务逻辑和IO操作，如果在handler中执行的业务逻辑特别慢会阻塞IO操作

处理方法

#### 一、在hander中启动新的线程来执行耗时操作

#### 二、告诉netty不要用之前定义的workGroup 来执行某些hander

```
// 事件处理线程
EventExecutorGroup eventExecutorGroup = new DefaultEventExecutorGroup(16);

ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup, workerGroup)
.channel(NioServerSocketChannel.class)
.childHandler(new ChannelInitializer<SocketChannel>() {
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
    ChannelPipeline p = ch.pipeline();
    p.addLast(new StringDecoder());
    p.addLast(new StringEncoder());
    
    // 如果你的业务逻辑是完全异步的，或者完成得非常快，那么就不需要这样做
    // 需要指定一个EventExecutorGroup，将用于执行ChannelHandler中的方法
    p.addLast(eventExecutorGroup, "String Echo", new StringEchoHandler());
    }
});


```





## 工具类

...
...

---

>大部分都是代码中的注释翻译过来的，没写完，有空再补充吧！



## 常用功能
### 心跳
```
//心跳检测，6分钟无心跳，触发关闭
public void initChannel(SocketChannel ch) throws Exception {
    ......
    pipeline.addLast( new IdleStateHandler( 6, 0, 0, TimeUnit.MINUTES ) );
    ......
}       
       
//6分钟没有读取到消息，会触发：userEventTriggered 方法

XxxxHandler

@Override
public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
	if (evt instanceof IdleStateEvent) {
		IdleStateEvent event = (IdleStateEvent) evt;
		if (event.state() == IdleState.READER_IDLE) {
			logger.warn("到达指定时间间隔没有收到心跳，关闭连接：{}", ctx.channel().remoteAddress());
			ctx.fireUserEventTriggered(evt);
			ctx.close();
		}
	} else {
		super.userEventTriggered(ctx, evt);
	}
}
```


