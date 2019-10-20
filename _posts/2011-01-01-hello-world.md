---
layout: post
title: "JAVA I/O模型介绍!"
description: "Netty 入门"
categories: [io]
tags: [io]
redirect_from:
  - /2019/10/20/
---
# IO的几种类型
  * 传统的阻塞I/O BIO
    > BIO是传统的网络编程所使用的通信方式，它的基本模型是Client/Server模型，也就是两个进程之间的相互通信，客户端通过连接操作向服务端监听的ip地址发起连接请求，通过三次握手协议建立连接。连接成功后，双方就能够通过网络套接字（socket）进行通信。是典型的一请求一应答通信模型。
  
    > 主要使用的类为:Socket、ServerSocket

    > 缺点： 当客户端并发访问量增加后，系统性能会急剧下降，并且随着并发访问量持续增大，会发生线程堆栈溢出、创建线程失败等问题，导致进程宕机或者僵死，不能对外提供服务。
  * 伪异步I/O
    > 伪异步I/O是采用线程池和任务队列的方式来实现的，相比于BIO来说，由于线程池可以设置消息队列的大小和最大线程数，因此他的资源是可控的，不会随客户端并发量提高而导致资源耗尽和宕机。

    > 缺点：由于伪异步阻塞I/O底层其实还是阻塞式IO，因此，当客户端请求写入较慢，服务端线程会一直等待写入完成，因此会导致服务端处理数据较慢，直到60超时，这时后续的I/O消息都会在消息队列中排队，当队列积满之后，后续入队列的操作将被阻塞，当队列阻塞后，Accptor线程被阻塞在线程池同步队列之后，新的客户端请求消息被拒绝，客户端会发生大量连接超时。用户会任务系统崩溃，无法接收新的请求消息。
  * 非阻塞I/O NIO
    > NIO指的是Non-block I/O，是在JDK 1.4中引入的，弥补了原来同步阻塞IO的不足，客户端发起的连接操作是异步的，不会阻塞后续的客户端请求，只需要向多路复用器注册OP_CONNECT，等待后续结果即可。JDK的Selector在linux等主流服务器上通过epoll实现，没有链接句柄的限制，意味着一个Selector线程可以同时处理成千上万客户端连接，并且性能不会随着客户端的增加而线性下降。因此，NIO非常适合做高性能、高负载的网络服务器。
    
    #### 缓冲区buffer ####
    > 在NIO的类库中加入了Buffer对象，这是新库与BIO的一个重要区别，在NIO库中，所有的数据都是缓冲区处理的，在读取数据时，它是直接读到缓冲区中的；在写入数据时，也是写入到缓冲区中，他的本质是一个数组，通常是一个字节数组（ButeBuffer），实时上，每一种java基本类型都对应了一种病缓冲区，具体如下：
    >> * ByteBuffer：字节缓冲区
    >> *  CharBuffer：字符缓冲区
    >> * ShortBuffer: 短整型缓冲区
    >> * IntBuffer：整形缓冲区
    >> * LongBuffer：长整形缓冲区
    >> * FloatBuffer： 浮点型缓冲区 

     #### 通道 Channel ####
    > Channel是要给通道，网络数据通过Channel读取和写入，通道与流不同之处在于通道是双向的，而流只在一个方向移动，必须是input或者output，而通道却可以读、写或者读写同时进行。NIO中主要使用ServerSocketChannel和 SocketChannel两个类
    
     #### 多路复用器 Selector ####
    > Selector会不断轮询注册在其上面的Channel，如果Channel发生读或者写的事件，这个Channel就会处于就绪状态，会被Selector检测出来，通过SelectionKey获取就绪的Channel集合，进行后续的I/O操作。一个Selector可以同属轮询多个Channel，由于JDK底层使用了epoll代替了select，所以并没有最大连接句柄1024/2048的限制，这也意味着一个Selector可以介入成千上万的客户端。

     > 主要使用的类库：ServerSockChannel、SocketChannel、Selector。
  * 非阻塞I/O NIO2.0 AIO
    > NIO 2.0引入了新的一部通道的概念，童工了异步文件通道和异步套接字通道的实现，并提供了一下两种方式获取操作结果
    >>* 通过java.util.concurrent.Future 类获取异步操作结果 
    >>* 通过执行异步操作的时候传入java.nio.channels
    
    > NIO 2.0的异步套接字通道是真正的异步非阻塞I/O，对应于UNIX网络编程中的 事件驱动I/O，他不再需要多路复用器（Selector）对注册的通道进行轮询即可实现异步读写。


