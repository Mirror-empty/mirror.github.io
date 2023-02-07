# netty基础入门

### NettyServer

服务端


```java
public class NettyServer {
    public static void main(String[] args) {
        new NettyServer().bing(7397);
    }

    private void bing(int port) {

        //配置服务器NIO线程组 相当于线程池 一个线程对应多个Channel ，一个Channel对应一个线程
        NioEventLoopGroup parentGroup  = new NioEventLoopGroup();

        NioEventLoopGroup childGroup  = new NioEventLoopGroup();

        try {
            // 核心操作是bind()方法，该方法新建一个serverSocketChannel，开始监听本地端口
            ServerBootstrap b = new ServerBootstrap();
            b.group(parentGroup,childGroup)
                    //非阻塞模式  Channel类型
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG,128)
                    .childHandler(new MyChannelInitializer());
            // 异步操作的结果
            ChannelFuture f = b.bind(port).sync();
            f.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            childGroup.shutdownGracefully();
            parentGroup.shutdownGracefully();
        }

    }
}

```

- EventLoopGroup: 不论是服务器端还是客户端, 都必须指定 EventLoopGroup. 在这个例子中, 指定了 NioEventLoopGroup, 表示一个 NIO 的EventLoopGroup, 不过服务器端需要指定两个 EventLoopGroup, 一个是 bossGroup, 用于处理客户端的连接请求; 另一个是 workerGroup, 用于处理与各个客户端连接的 IO 操作.

### MyServerHandler


- ByteBuf是netty的Server与Client之间通信的数据传输载体(Netty的数据容器)，它提供了一个byte数组(byte[])的抽象视图，既解决了JDK API的局限性，又为网络应用程序的开发者提供了更好的API


- ByteBuf工作机制:ByteBuf维护了两个不同的索引，一个用于读取，一个用于写入。readerIndex和writerIndex的初始值都是0，当从ByteBuf中读取数据时，它的readerIndex将会被递增(它不会超过writerIndex)，当向ByteBuf写入数据时，它的writerIndex会递增。

- 名称以readXXX或者writeXXX开头的ByteBuf方法，会推进对应的索引，而以setXXX或getXXX开头的操作不会。

- 在读取之后，0～readerIndex的就被视为discard的，调用discardReadBytes方法，可以释放这部分空间，它的作用类似ByteBuffer的compact()方法。

- readerIndex和writerIndex之间的数据是可读取的，等价于ByteBuffer的position和limit之间的数据。writerIndex和capacity之间的空间是可写的，等价于ByteBuffer的limit和capacity之间的可用空间。


```java
public class MyServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        // 接收msg消息
        ByteBuf buf = (ByteBuf) msg;
        byte[] msgByte = new byte[buf.readableBytes()];
        buf.readBytes(msgByte);
        System.out.print(new Date() + "接收到消息：");
        System.out.println(new String(msgByte, Charset.forName("GBK")));
    }
}

```

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


![图 1](../../../images/ee5608d11740cd6e65c89a3e15bac877791feaa9414c96b2ae17f0c6dd2a07b2.png)  



![图 2](ee5608d11740cd6e65c89a3e15bac877791feaa9414c96b2ae17f0c6dd2a07b2.png)  

![图 3](ee5608d11740cd6e65c89a3e15bac877791feaa9414c96b2ae17f0c6dd2a07b2.png)  

![图 4](../../../image/ee5608d11740cd6e65c89a3e15bac877791feaa9414c96b2ae17f0c6dd2a07b2.png)  

![图 5](../image/ee5608d11740cd6e65c89a3e15bac877791feaa9414c96b2ae17f0c6dd2a07b2.png)  


![图 6](../image/ee5608d11740cd6e65c89a3e15bac877791feaa9414c96b2ae17f0c6dd2a07b2.png)  

![图 7](../image/2023.02.07.%7Bhh%7D.png)  


