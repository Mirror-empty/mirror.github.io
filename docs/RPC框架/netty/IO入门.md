# IO入门

>Java BIO[Blocking I/O] 同步阻塞I/O模式

BIO 全程Block-IO是一种同步且阻塞的通信模式。是一个比较传统的通信方式，使用方便。但是并发处理能力低，通信耗时，依赖网速。

>Java NIO[New I/O] 同步非阻塞模式

- 非阻塞的通信方式
- 面向流的I/O系统一次一个字节低处理数据。
- 面向块的I/O系统以块的形式处理数据。

>Java AIO[Asynchronous I/O] 异步非阻塞I/O模型

Java AIO，全程 Asynchronous IO，是异步非阻塞的IO。是一种非阻塞异步的通信模式。在NIO的基础上引入了新的异步通道的概念，并提供了异步文件通道和异步套接字通道的实现。


### AIO

1. AioServer 服务端 来接收消息 
2. 创建线程一直监听通道，（创建线程组是异步的）

```java
 //此方法启动异步操作以接受对该通道的套接字进行的连接。 handler参数是一个完成处理程序，在连接被接
    serverSocketChannel.accept(this,new AioServerChannelInitializer());
```
3. 在AioServerChannelInitializer中会实现 将通道的字节序列读到缓冲区 然后将缓冲区的消息解析出来


```java
public class AioServer extends Thread{

    /**
     * 用于面向流的侦听套接字的异步通道。
     * 通过调用此类的open方法创建异步服务器套接字通道。
     * 新创建的异步服务器套接字通道已打开但尚未绑定。 它可以绑定到本地地址，并配置为通过调用bind方法监听连接
     *  监听作用
     */
    private AsynchronousServerSocketChannel serverSocketChannel;

    @Override
    public void run() {
        //                                    用于资源共享的一组异步通道。
        // 异步通道组封装了处理绑定到组的asynchronous channels发起的I / O操作完成所需的机制
        // 创建异步组
        // 创建具有给定线程池的异步通道组，根据需要创建新线程 意思是监听用线程组
        try {
            serverSocketChannel = AsynchronousServerSocketChannel.open(AsynchronousChannelGroup.withCachedThreadPool(
                    Executors.newCachedThreadPool(), 10
            ));
            //将通道的套接字绑定到本地地址，并配置套接字以监听连接。没设置ip 本地
            serverSocketChannel.bind(new InetSocketAddress(7397));
            CountDownLatch latch = new CountDownLatch(1);
            //此方法启动异步操作以接受对该通道的套接字进行的连接。 handler参数是一个完成处理程序，在连接被接
            serverSocketChannel.accept(this,new AioServerChannelInitializer());
            try {
                latch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

```

```java
// 消费处理结果程序
public class AioServerChannelInitializer extends ChannelInitializer {


    protected void initChannel(AsynchronousSocketChannel channel) throws Exception {
        //从通道将字节序列读到缓冲区
        channel.read(ByteBuffer.allocate(1024), 10, TimeUnit.SECONDS, null, new AioServerHandler(channel, Charset.forName("GBK")));
    }

}
```

```java
public abstract class ChannelAdapter implements CompletionHandler<Integer,Object> {

    private AsynchronousSocketChannel channel;

    /**
     * 16位Unicode code 和字节序列的命令映射
     */
    private Charset charset;

    public ChannelAdapter(AsynchronousSocketChannel channel,Charset charset){
        this.channel = channel;
        this.charset = charset;
        if (channel.isOpen()) {
            channelActive(new ChannelHandler(channel, charset));
        }
    }



    @Override
    public void completed(Integer result, Object attachment) {

        final ByteBuffer buffer = ByteBuffer.allocate(1024);
        final long timeout = 60 * 60L;
        channel.read(buffer, timeout, TimeUnit.SECONDS, null, new CompletionHandler<Integer, Object>() {
            @Override
            public void completed(Integer result, Object attachment) {
                if (result == -1) {
                    try {
                        channelInactive(new ChannelHandler(channel, charset));
                        channel.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                    return;
                }
                buffer.flip();
                // 读取到消息
                channelRead(new ChannelHandler(channel, charset), charset.decode(buffer));
                buffer.clear();
                channel.read(buffer, timeout, TimeUnit.SECONDS, null, this);
            }

            @Override
            public void failed(Throwable exc, Object attachment) {
                exc.printStackTrace();
            }
        });


    }
```
```java
public class ChannelHandler {

    private AsynchronousSocketChannel channel;
    private Charset charset;

    public ChannelHandler(AsynchronousSocketChannel channel, Charset charset) {
        this.channel = channel;
        this.charset = charset;
    }

    public void writeAndFlush(Object msg) {
    //    获取字节
        byte[] bytes = msg.toString().getBytes();
        // 创建一个字节缓冲区并设置其容量
        ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
        // 此方法将给定源字节数组的整个内容传输到此缓冲区。 调用此方法的形式为dst.put(a)的行为方式与调用完全相同
        writeBuffer.put(bytes);
        // 翻转整个缓冲区
        writeBuffer.flip();
        // 该方法启动异步写入操作，以从给定的缓冲区向该通道写入字节序列
        channel.write(writeBuffer);
    }
```



### NIO

主要是有多路复用

```java
public class NioClient {

    public static void main(String[] args) throws IOException {
        /**
         * Selector (选择器) 多路复用的概念
         * 多路是指：服务端同时监听多个“端口”的情况，每个端口都要监听多个客户端的连接。
         *
         */
        // 这里是Selector选择器的获取
        Selector selector = Selector.open();

        // 创建一个通道对象
        SocketChannel socketChannel = SocketChannel.open();

        // 设置为非阻塞状态
        socketChannel.configureBlocking(false);

        boolean connect = socketChannel.connect(new InetSocketAddress("192.168.10.46", 7397));
        if (connect) {
            // 注册Channel 到Selector 交给选择器来处理
            //register()方法的第二个参数：是一个int值，意思是在通过Selector监听Channel时对什么事件感兴趣。
            /**
             * 而且可以使用SelectionKey的四个常量表示：
             *
             * 连接就绪–常量：SelectionKey.OP_CONNECT
             *
             * 接收就绪–常量：SelectionKey.OP_ACCEPT (ServerSocketChannel在注册时只能使用此项)
             *
             * 读就绪–常量：SelectionKey.OP_READ
             *
             * 写就绪–常量：SelectionKey.OP_WRITE
             */
            socketChannel.register(selector, SelectionKey.OP_READ);
        } else  {
            socketChannel.register(selector, SelectionKey.OP_CONNECT);
        }
        new NioClientHandler(selector, Charset.forName("GBK")).start();
    }
}
```

去判断在哪个端 然后判断是否可以被获取 打印出来

```java
public abstract class ChannelAdapter extends Thread {

    private Selector selector;

    private ChannelHandler channelHandler;
    private Charset charset;

    public ChannelAdapter(Selector selector, Charset charset) {
        this.selector = selector;
        this.charset = charset;
    }

    @Override
    public void run() {
        while (true) {
            try {
                // 服务器等待客户端的连接
                selector.select(1000);
                // 获取已连接的所有通道
                Set<SelectionKey> selectedKeys = selector.selectedKeys();

                Iterator<SelectionKey> it = selectedKeys.iterator();
                // 一个令牌代表一个连接
                SelectionKey key = null;
                while (it.hasNext()) {
                    key = it.next();
                    it.remove();
                    handleInput(key);
                }
            } catch (Exception ignore) {
            }
        }
    }
    private void handleInput(SelectionKey key) throws IOException {
        // 判断密钥是否有效
        if (!key.isValid()) {
            return;
        }

        // 客户端SocketChannel SelectionKey是Channel的封装 这里是获取channel
        Class<?> superclass = key.channel().getClass().getSuperclass();
        if (superclass == SocketChannel.class){
            SocketChannel socketChannel = (SocketChannel) key.channel();
            // 判断此通道是否完成
            if (key.isConnectable()) {
                if (socketChannel.finishConnect()) {
                    channelHandler = new ChannelHandler(socketChannel, charset);
                    channelActive(channelHandler);
                    socketChannel.register(selector, SelectionKey.OP_READ);
                } else {
                    System.exit(1);
                }
            }
        }

        // 服务端ServerSocketChannel
        if (superclass == ServerSocketChannel.class){
            if (key.isAcceptable()) {
                ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
                SocketChannel socketChannel = serverSocketChannel.accept();
                socketChannel.configureBlocking(false);
                socketChannel.register(selector, SelectionKey.OP_READ);

                channelHandler = new ChannelHandler(socketChannel, charset);
                channelActive(channelHandler);
            }
        }

        // 这个通道是否可以被获取
        if (key.isReadable()) {
            SocketChannel socketChannel = (SocketChannel) key.channel();
            ByteBuffer readBuffer = ByteBuffer.allocate(1024);
            int readBytes = socketChannel.read(readBuffer);
            if (readBytes > 0) {
                readBuffer.flip();
                byte[] bytes = new byte[readBuffer.remaining()];
                readBuffer.get(bytes);
                channelRead(channelHandler, new String(bytes, charset));
            } else if (readBytes < 0) {
                key.cancel();
                socketChannel.close();
            }
        }
    }

    // 链接通知抽象类
    public abstract void channelActive(ChannelHandler ctx);

    // 读取消息抽象类
    public abstract void channelRead(ChannelHandler ctx, Object msg);

}
```
