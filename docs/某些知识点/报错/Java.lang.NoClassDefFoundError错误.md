# Java.lang.NoClassDefFoundError错误

这是在第一次用doubbo时，客户端去远程调用服务端rpc接口时，我们需要去rpc中的模块打包，如果点击的是root跟模块下打开会把所有模块都会打包，这种方法是错误的。

需要单独对rpc模块打包，反正就是因为一些原因触发了这个错误。

这种错误的几个原因：

1. 对应的Class在java的classpath中不可用
2. 你可能用jar命令运行你的程序，但类并没有在jar文件的manifest文件中的classpath属性中定义
3. 可能程序的启动脚本覆盖了原来的classpath环境变量
4. 因为NoClassDefFoundError是java.lang.LinkageError的一个子类，所以可能由于程序依赖的原生的类库不可用而导致
5. 检查日志文件中是否有java.lang.ExceptionInInitializerError这样的错误，NoClassDefFoundError有可能是由于静态初始化失败导致的
6. 如果你工作在J2EE的环境，有多个不同的类加载器，也可能导致NoClassDefFoundError
7. 跨进程调用  导致找不到那个类【这一点是经常被忽略的，很坑】

![zxHa2n.jpg](https://s1.ax1x.com/2022/12/26/zxHa2n.jpg)


然后下面还报了ClassNotFoundException错误

而ClassNotFoundException是在编译的时候在classpath中找不到对应的类而发生的错误。


[参考](https://cloud.tencent.com/developer/article/1459447)