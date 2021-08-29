---
layout: post
title: java网络IO模型详解
date: '2019-09-14 16:02'
categories: Java
tags: [Java]
description: java网络IO模型详解
comments: true
---

java网络IO模型详解

目录
[TOC]

# 一、背景
在互联网的时代下，绝大部分数据都是通过网络来进行获取的。在服务端的架构中，绝大部分数据也是通过网络来进行交互的。作为服务端的开发工程师来说，都会进行一系列服务设计、开发以及能力开放，而服务能力开放也是需要通过网络来完成的，因此对网络编程以及网络IO模型都不会太陌生。由于有很多优秀的框架（比如Netty、HSF、Dubbo、Thrift等）已经把底层网络IO给封装了，通过提供的API能力或者配置就能完成想要的服务能力开发，因此大部分工程师对网络IO模型的底层不够了解。本文系统的讲解了Linux内核的IO模型、Java网络IO模型以及两者之间的关系，让大家在完成本职工作的情况下加深对这一块底层原理的掌握，知其然也知其所以然，在异常定位、性能优化等方面都可以有自己独到的见解以及理论依据。

# 二、什么是IO
我们都知道在Linux的世界，一切皆文件。而文件是什么呢，文件就是一串二进制流。不管socket、FIFO、管道还是终端，对我们来说，一切都是流。在信息的交换过程中，

我们都是对这些流进行数据收发操作，简称为I/O操作（input and output）。往流中读取数据，系统调用read，写入数据，系统调用write。

通常用户进程的一个完整的IO分为两个阶段：

1.以磁盘IO为例：


2.以网络IO为例：


操作系统和驱动程序运行在内核空间，应用程序运行在用户空间，两者不能使用指针传递数据，因为Linux使用的虚拟内存机制，必须通过系统调用请求内核来完成IO动作。

IO有内存IO、网络IO和磁盘IO三种，通常我们说的IO指的是后两者，本篇文章重点讲解网络IO，由于java应用程序的IO操作依赖操作系统内核才能完成，因此选了Linux内核的IO模型进行详细讲解，目的是为了更好的理解java的网络IO模型。

# 三、Linux内核的IO模型
## 3.1 概念说明
### 3.1.1 用户空间和内核空间
现在操作系统都是采用虚拟存储器，那么对32位操作系统而言，它的寻址空间（虚拟存储空间）为4G（2的32次方）。操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证内核的安全，用户进程不能直接操作内核（kernel），操作系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。针对linux操作系统而言，将最高的1G字节（从虚拟地址0xC0000000到0xFFFFFFFF），供内核使用，称为内核空间，而将较低的3G字节（从虚拟地址0x00000000到0xBFFFFFFF），供各个进程使用，称为用户空间。 

### 3.1.2 进程切换
为了控制进程的执行，内核必须有能力挂起正在CPU上运行的进程，并恢复以前挂起的某个进程的执行。这种行为被称为进程切换。因此可以说，任何进程都是在操作系统内核的支持下运行的，是与内核紧密相关的。

从一个进程的运行转到另一个进程上运行，这个过程中经过下面这些变化：

保存处理机上下文，包括程序计数器和其他寄存器

更新PCB（进程控制块）信息

把进程的PCB（进程控制块）移入相应的队列，如就绪、在某事件阻塞等队列

选择另一个进程执行，并更新其PCB（进程控制块）

更新内存管理的数据结构

恢复处理机上下文

### 3.1.3 进程的阻塞
正在执行的进程，由于期待的某些事件未发生，如请求系统资源失败、等待某种操作的完成、新数据尚未到达或无新工作做等，则由系统自动执行阻塞原语(Block)，使自己由运行状态变为阻塞状态。可见，进程的阻塞是进程自身的一种主动行为，也因此只有处于运行态的进程（获得CPU），才可能将其转为阻塞状态。当进程进入阻塞状态，是不占用CPU资源的。

### 3.1.4 文件描述符
文件描述符（File descriptor）是计算机科学中的一个术语，是一个用于表述指向文件的引用的抽象化概念。文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

### 3.1.5 缓存 IO
缓存 IO 又被称作标准 IO，大多数文件系统的默认 IO 操作都是缓存 IO。在 Linux 的缓存 IO 机制中，操作系统会将 IO 的数据缓存在文件系统的页缓存（ page cache ）中，也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。

缓存 IO 的缺点：数据在传输过程中需要在应用程序地址空间和内核空间进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是非常大的。 

## 3.2 同步阻塞
用户空间的应用程序执行一个系统调用（recvform），这会导致应用程序阻塞，什么也不干，直到数据准备好，并且将数据从内核复制到用户进程，最后进程再处理数据，在等待数据到处理数据的两个阶段，整个进程都被阻塞，不能处理别的网络IO。调用应用程序处于一种不再消费 CPU 而只是简单等待响应的状态，因此从处理的角度来看，这是非常有效的。

这也是最简单的IO模型，在通常fd较少、就绪很快的情况下使用是没有问题的。


## 3.3 同步非阻塞
非阻塞的recvform系统调用调用之后，进程并没有被阻塞，内核马上返回给进程，如果数据还没准备好，此时会返回一个error。进程在返回之后，可以干点别的事情，然后再发起recvform系统调用。重复上面的过程，循环往复的进行recvform系统调用。这个过程通常被称之为轮询。轮询检查内核数据，直到数据准备好，再拷贝数据到进程，进行数据处理。需要注意，拷贝数据整个过程，进程仍然是属于阻塞的状态。

这种方式在编程中对socket设置O_NONBLOCK即可。但此方式仅仅针对网络IO有效，对磁盘IO并没有作用。因为本地文件IO就没有被认为是阻塞，我们所说的网络IO的阻塞是因为网路IO有无限阻塞的可能，而本地文件除非是被锁住，否则是不可能无限阻塞的，因此只有锁这种情况下，O_NONBLOCK才会有作用。而且，磁盘IO时要么数据在内核缓冲区中直接可以返回，要么需要调用物理设备去读取，这时候进程的其他工作都需要等待。因此，后续的IO复用和信号驱动IO对文件IO也是没有意义的。


## 3.4 IO复用
IO复用，也叫多路IO就绪通知。这是一种进程预先告知内核的能力，让内核发现进程指定的一个或多个IO条件就绪了，就通知进程。使得一个进程能在一连串的事件上等待。

IO复用的实现方式目前主要有select、poll和epoll。

select和poll的原理基本相同：

注册待侦听的fd（文件描述符）,这里的fd（文件描述符）创建时最好使用非阻塞

每次调用都去检查这些fd（文件描述符）的状态，当有一个或者多个fd（文件描述符）就绪的时候返回

返回结果中包括已就绪和未就绪的fd

相比select，poll解决了单个进程能够打开的文件描述符数量有限制这个问题：select受限于FD_SIZE的限制，如果修改则需要修改这个宏重新编译内核；而poll通过一个pollfd数组向内核传递需要关注的事件，避开了文件描述符数量限制。

此外，select和poll共同具有的一个很大的缺点就是包含大量fd的数组被整体复制于用户态和内核态地址空间之间，开销会随着fd（文件描述符）数量增多而线性增大。

epoll的出现，解决了select、poll的缺点：

基于事件驱动的方式，避免了每次都要把所有fd（文件描述符）都扫描一遍

epoll_wait只返回就绪的fd（文件描述符）

epoll使用nmap内存映射技术避免了内存复制的开销

epoll的fd（文件描述符）数量上限是操作系统的最大文件句柄数目,这个数目一般和内存有关，通常远大于1024

目前，epoll是Linux2.6下最高效的IO复用方式，也是Nginx、Node的IO实现方式。而在freeBSD下，kqueue是另一种类似于epoll的IO复用方式。

此外，对于IO复用还有一个水平触发和边缘触发的概念：

水平触发：当就绪的fd（文件描述符）未被用户进程处理后，下一次查询依旧会返回，这是select和poll的触发方式

边缘触发：无论就绪的fd（文件描述符）是否被处理，下一次不再返回。理论上性能更高，但是实现相当复杂，并且任何意外的丢失事件都会造成请求处理错误。epoll默认使用水平触发，通过相应选项可以使用边缘触发


## 3.5 信号驱动
首先我们允许Socket进行信号驱动IO,并安装一个信号处理函数，进程继续运行并不阻塞。当数据准备好时，进程会收到一个SIGIO信号，可以在信号处理函数中调用I/O操作函数处理数据。流程如下：

开启套接字信号驱动IO功能

系统调用sigaction执行信号处理函数（非阻塞，立刻返回）

数据就绪，生成sigio信号，通过信号回调通知应用来读取数据

此种io方式存在的一个很大的问题：Linux中信号队列是有限制的，如果超过这个数字问题就无法读取数据


## 3.6 异步非阻塞
异步io流程如下所示：

当用户线程调用了aio_read系统调用，立刻就可以开始去做其它的事，用户线程不阻塞

内核（kernel）就开始了IO的第一个阶段：准备数据。当kernel一直等到数据准备好了，它就会将数据从kernel内核缓冲区，拷贝到用户缓冲区（用户内存）

kernel会给用户线程发送一个信号（signal），或者回调用户线程注册的回调接口，告诉用户线程read操作完成了

用户线程读取用户缓冲区的数据，完成后续的业务操作

相对于同步IO，异步IO不是顺序执行。用户进程进行aio_read系统调用之后，无论内核数据是否准备好，都会直接返回给用户进程，然后用户态进程可以去做别的事情。等到数据准备好了，内核直接复制数据给进程，然后从内核向进程发送通知。IO两个阶段，进程都是非阻塞的。

对比信号驱动IO，异步IO的主要区别在于：信号驱动由内核告诉我们何时可以开始一个IO操作(数据在内核缓冲区中)，而异步IO则由内核通知IO操作何时已经完成(数据已经在用户空间中)。

异步IO又叫做事件驱动IO，在Unix中，POSIX1003.1标准为异步方式访问文件定义了一套库函数，定义了AIO的一系列接口。使用aio_read或者aio_write发起异步IO操作，使用aio_error检查正在运行的IO操作的状态。但是其实现没有通过内核而是使用了多线程阻塞。此外，还有Linux自己实现的Native AIO，依赖两个函数：io_submit和io_getevents，虽然io是非阻塞的，但仍需要主动去获取读写的状态。

目前Linux中AIO的内核实现只对文件IO有效，如果要实现真正的AIO，需要用户自己来实现。目前有很多开源的异步IO库，例如libevent、libev、libuv。


# 四、Java网络IO模型
## 4.1 BIO
BIO是一个典型的网络编程模型，是通常我们实现一个服务端程序的方法，对应Linux内核的同步阻塞IO模型，发送数据和接收数据的过程如下所示：


步骤如下：

主线程accept请求

请求到达，创建新的线程来处理这个套接字，完成对客户端的响应

主线程继续accept下一个请求

发送过程：

应用程序调用系统调用，将数据发送给socket

socket检查数据类型，调用相应的send函数

send函数检查socket状态、协议类型，传给传输层

tcp/udp（传输层协议）为这些数据创建数据结构，加入协议头部，比如端口号、检验和。

ip（网络层协议）添加ip头，比如ip地址、检验和，如果数据包大小超过了mtu（最大数据包大小），则分片；ip将这些数据包传给链路层写到网卡队列

网卡调用相应中断驱动程序，发送到网络

接收过程：

数据包从网络到达网卡，网卡接收帧，放入网卡buffer，再向系统发送中断请求

cpu调用相应中断函数，这些中断处理程序在网卡驱动中

中断处理函数从网卡读入内存，交给链路层

链路层将包放入自己的队列，置软中断标志位

进程调度器看到了标志位，调度相应进程

该进程将包从队列取出，与相应协议匹配，一般为ip协议，再将包传递给该协议接收函数

ip对包进行错误检测，如果无错则进行路由。路由结果：packet被转发或者继续向上层传递

如果发往本机，进入链路层

链路层再进行错误侦测，查找相应端口关联socket，包被放入相应socket接收队列

socket唤醒拥有该socket的进程，进程从系统调用read中返回，将数据拷贝到自己的buffer，返回用户态

服务端处理伪代码如下所示：
```
代码块
{
 ExecutorService executor = Excutors.newFixedThreadPollExecutor(100);//线程池
​
 ServerSocket serverSocket = new ServerSocket();
 serverSocket.bind(8088);
 while(!Thread.currentThread.isInturrupted()){//主线程死循环等待新连接到来
     Socket socket = serverSocket.accept();
     executor.submit(new ConnectIOnHandler(socket));//为新的连接创建新的线程
}
​
class ConnectIOnHandler extends Thread{
    private Socket socket;
    public ConnectIOnHandler(Socket socket){
       this.socket = socket;
    }
    public void run(){
      while(!Thread.currentThread.isInturrupted()&&!socket.isClosed()){死循环处理读写事件
          String someThing = socket.read()....//读取数据
          if(someThing!=null){
             ......//处理数据
             socket.write()....//写数据
          }
      }
    }
}
```
这是经典的一个连接对应一个线程的模型，之所以使用多线程，主要原因在于socket.accept()、socket.read()、socket.write()三个主要函数都是同步阻塞的，当一个连接在处理I/O的时候，系统是阻塞的，如果是单线程的话必然就阻塞，但CPU是被释放出来的，开启多线程，就可以让CPU去处理更多的事情。其实这也是所有使用多线程的本质：利用多核，当I/O阻塞时，但CPU空闲的时候，可以利用多线程使用CPU资源。

现在的多线程一般都使用线程池，可以让线程的创建和回收成本相对较低。在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的I/O并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。

不过，这个模型最本质的问题在于，严重依赖于线程。但线程是很"贵"的资源，主要表现在：

线程的创建和销毁成本很高，在Linux这样的操作系统中，线程本质上就是一个进程。创建和销毁都是重量级的系统函数。

线程本身占用较大内存，像Java的线程栈，一般至少分配512K～1M的空间，如果系统中的线程数过千，恐怕整个JVM的内存都会被吃掉一半。

线程的切换成本是很高的。操作系统发生线程切换的时候，需要保留线程的上下文，然后执行系统调用。如果线程数过高，可能执行线程切换的时间甚至会大于线程执行的时间，这时候带来的表现往往是系统load偏高、CPU sy使用率特别高（超过20%以上)，导致系统几乎陷入不可用的状态。容易造成锯齿状的系统负载。因为系统负载是用活动线程数或CPU核心数，一旦线程数量高但外部网络环境不是很稳定，就很容易造成大量请求的结果同时返回，激活大量阻塞线程从而使系统负载压力过大。

所以，当面对十万甚至百万级连接的时候，传统的BIO模型是无能为力的。随着移动端应用的兴起和各种网络游戏的盛行，百万级长连接日趋普遍，此时，必然需要一种更高效的I/O处理模型。

## 4.2 NIO
JDK1.4开始引入了NIO类库，这里的NIO指的是Non-blcok IO，主要是使用Selector多路复用器来实现。Selector在Linux等主流操作系统上是通过IO复用epoll实现的。

NIO的实现流程，类似于select：

创建ServerSocketChannel监听客户端连接并绑定监听端口，设置为非阻塞模式

创建Reactor线程，创建多路复用器(Selector)并启动线程

将ServerSocketChannel注册到Reactor线程的Selector上，监听accept事件

Selector在线程run方法中无线循环轮询准备就绪的Key

Selector监听到新的客户端接入，处理新的请求，完成tcp三次握手，建立物理连接

将新的客户端连接注册到Selector上，监听读操作，读取客户端发送的网络消息

客户端发送的数据就绪则读取客户端请求，进行处理

简单处理模型是用一个单线程死循环选择就绪的事件，会执行系统调用（Linux 2.6之前是select、poll，2.6之后是epoll，Windows是IOCP），还会阻塞的等待新事件的到来。新事件到来的时候，会在selector上注册标记位，标示可读、可写或者有连接到来，简单处理模型的伪代码如下所示：

代码块
   interface ChannelHandler{
      void channelReadable(Channel channel);
      void channelWritable(Channel channel);
   }
   class Channel{
      Socket socket;
      Event event;//读，写或者连接
   }
​
   //IO线程主循环:
   class IoThread extends Thread{
       public void run(){
           Channel channel;
           while(channel=Selector.select()){//选择就绪的事件和对应的连接
               if(channel.event==accept){
                  registerNewChannelHandler(channel);//如果是新连接，则注册一个新的读写处理器
               }
               if(channel.event==write){
                  getChannelHandler(channel).channelWritable(channel);//如果可以写，则执行写事件
               }
               if(channel.event==read){
                   getChannelHandler(channel).channelReadable(channel);//如果可以读，则执行读事件
               }
          }
      }
      Map<Channel，ChannelHandler> handlerMap;//所有channel的对应事件处理器
  }
​
NIO由原来的阻塞读写（占用线程）变成了单线程轮询事件，找到可以进行读写的网络描述符进行读写。除了事件的轮询是阻塞的（没有可干的事情必须要阻塞），剩余的I/O操作都是纯CPU操作，没有必要开启多线程。并且由于线程的节约，连接数大的时候因为线程切换带来的问题也随之解决，进而为处理海量连接提供了可能。

但是目前主流服务器都是多核处理器，简单处理模型没有利用到多核处理器的优势，为了充分利用多核处理器，出现了优化线程处理模型，根据IO处理的几个阶段，分成不同角色的处理线程：

事件分发器，单线程选择就绪的事件

I/O处理器，包括connect、read、write等，这种纯CPU操作，一般开启CPU核数个线程就可以

业务线程，在处理完I/O后，业务一般还会有自己的业务逻辑，有的还会有其他的阻塞I/O，如DB操作，RPC等。只要有阻塞，就需要单独的线程


## 4.3 AIO
JDK1.7引入NIO2.0，提供了异步文件通道和异步套接字通道的实现。其底层在windows上是通过IOCP实现，在Linux上是通过IO复用epoll来模拟实现的(UnixAsynchronousServerSocketChannelImpl.java)。

在JAVA NIO框架中，我们说到了一个重要概念“selector”（选择器）。它负责代替应用查询中所有已注册的通道到操作系统中进行IO事件轮询、管理当前注册的通道集合，定位发生事件的通道等操作。但是在JAVA AIO框架中，由于应用程序不是“轮询”方式，而是订阅-通知方式，所以不再需要“selector”（选择器）了，改由channel通道直接到操作系统注册监听 。

JAVA AIO框架中，只实现了两种网络IO通道“AsynchronousServerSocketChannel”（服务器监听通道）、“AsynchronousSocketChannel”（socket套接字通道）。但是无论哪种通道他们都有独立的fileDescriptor（文件标识符）、attachment（附件，附件可以使任意对象，类似“通道上下文”），并被独立的SocketChannelReadHandle类实例引用 。过程如下所示：

创建AsynchronousServerSocketChannel，绑定监听端口

调用AsynchronousServerSocketChannel的accpet方法，传入自己实现的CompletionHandler，包括上一步，都是非阻塞的

连接传入，回调CompletionHandler的completed方法，在里面，调用AsynchronousSocketChannel的read方法，传入负责处理数据的CompletionHandler

数据就绪，触发负责处理数据的CompletionHandler的completed方法，继续做下一步处理即可

写入操作类似，也需要传入CompletionHandler

代码块
```
// 客户端代码
package JAVA_IO;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousSocketChannel;
 
public class AIOClient {
    public static void main(String... args) throws Exception {
        AsynchronousSocketChannel client = AsynchronousSocketChannel.open();
        client.connect(new InetSocketAddress("localhost", 8088)).get();
        client.write(ByteBuffer.wrap("123456789".getBytes()));
        Thread.sleep(10000);
    }
}
​
// 服务端代码
package JAVA_IO;
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
import java.nio.charset.Charset;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;
 
public class AIOServer {
    public final static int PORT = 8088;
    private AsynchronousServerSocketChannel server;
 
    public AIOServer() throws IOException {
        server = AsynchronousServerSocketChannel.open().bind(
                new InetSocketAddress(PORT));
    }
 
    public void startWithFuture() throws InterruptedException,
            ExecutionException, TimeoutException {
        while (true) {// 循环接收客户端请求
            Future<AsynchronousSocketChannel> future = server.accept();
            AsynchronousSocketChannel socket = future.get();// get() 是为了确保 accept 到一个连接
            handleWithFuture(socket);
        }
    }
    public void handleWithFuture(AsynchronousSocketChannel channel) throws InterruptedException, ExecutionException, TimeoutException {
        ByteBuffer readBuf = ByteBuffer.allocate(2);
        readBuf.clear();
 
        while (true) {// 一次可能读不完
            //get 是为了确保 read 完成，超时时间可以有效避免DOS攻击，如果客户端一直不发送数据，则进行超时处理
            Integer integer = channel.read(readBuf).get(10, TimeUnit.SECONDS);
            System.out.println("read: " + integer);
            if (integer == -1) {
                break;
            }
            readBuf.flip();
            System.out.println("received: " + Charset.forName("UTF-8").decode(readBuf));
            readBuf.clear();
        }
    }
 
    public void startWithCompletionHandler() throws InterruptedException,
            ExecutionException, TimeoutException {
        server.accept(null,
                new CompletionHandler<AsynchronousSocketChannel, Object>() {
                    public void completed(AsynchronousSocketChannel result, Object attachment) {
                        server.accept(null, this);// 接收客户端连接
                        handleWithCompletionHandler(result);
                    }
 
                    @Override
                    public void failed(Throwable exc, Object attachment) {
                        exc.printStackTrace();
                    }
                });
    }
 
    public void handleWithCompletionHandler(final AsynchronousSocketChannel channel) {
        try {
            final ByteBuffer buffer = ByteBuffer.allocate(4);
            final long timeout = 10L;
            channel.read(buffer, timeout, TimeUnit.SECONDS, null, new CompletionHandler<Integer, Object>() {
                @Override
                public void completed(Integer result, Object attachment) {
                    System.out.println("read:" + result);
                    if (result == -1) {
                        try {
                            channel.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        return;
                    }
                    buffer.flip();
                    System.out.println("received message:" + Charset.forName("UTF-8").decode(buffer));
                    buffer.clear();
                    channel.read(buffer, timeout, TimeUnit.SECONDS, null, this);
                }
 
                @Override
                public void failed(Throwable exc, Object attachment) {
                    exc.printStackTrace();
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
 
    public static void main(String args[]) throws Exception {
        // new AIOServer().startWithFuture();
        new AIOServer().startWithCompletionHandler();
        Thread.sleep(100000);
    }
}
```
## 4.4 对比
同步阻塞IO

伪异步IO

NIO

AIO

客户端数目 ：IO线程

1 : 1

m : n

m : 1

m : 0

IO模型

同步阻塞IO

同步阻塞IO

同步非阻塞IO

异步非阻塞IO

吞吐量

低

中

高

高

编程复杂度

简单

简单

比AIO复杂

复杂

# 五、结语
我们在做网络编程时也会经历几个阶段：

阶段一、通过api文档以及说明能够完成功能开发，没有什么疑问，也就是“看山是山，看水是水”的阶段。

阶段二、使用api的时候总会想着为什么这个api能实现想要的功能，它到底是怎样实现的，也就是“看山不是山，看水不是水”的阶段。

阶段三、知道了每个api的功能以及底层实现原理，知道了怎么做才是最优的方式，没有什么疑问，也就是“看山还是山，看水还是水”的阶段。