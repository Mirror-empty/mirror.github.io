# IO入门

>Java BIO[Blocking I/O] 同步阻塞I/O模式

BIO 全程Block-IO是一种同步且阻塞的通信模式。是一个比较传统的通信方式，使用方便。但是并发处理能力低，通信耗时，依赖网速。

>Java NIO[New I/O] 同步非阻塞模式

- 非阻塞的通信方式
- 面向流的I/O系统一次一个字节低处理数据。
- 面向块的I/O系统以块的形式处理数据。

>Java AIO[Asynchronous I/O] 异步非阻塞I/O模型

Java AIO，全程 Asynchronous IO，是异步非阻塞的IO。是一种非阻塞异步的通信模式。在NIO的基础上引入了新的异步通道的概念，并提供了异步文件通道和异步套接字通道的实现。





