# 搭建erp并使用nacos作为注册中心

### 使用DDD分层搭建

application应用层定义接口，串联整个项目逻辑的实现

domain领域层做实现，将大的的逻辑进行拆分，一个一个的实现小的逻辑。

infrastructure基础层，定义实体类，持久层

interfaces接口层对外暴露

将rpc打包出来，被服务接口被其他引入


```java
java.net.ConnectException: Connection refused: connect
```

doubbo消费端的端口不能和生产端一致。


### 使用nacos作为注册中心

注册中心是什么？

最开始是去分配IP，端口的，然后去涉及其它的性能

[参考官网配置](https://nacos.io/zh-cn/docs/use-nacos-with-dubbo.html)


访问提供方和消费方都需要去进行配置

```yml
# nacos 配置中心
nacos:
  discovery:
    access-key: 127.0.0.1.8848

# Dubbo 广播方式配置
dubbo:
  application:
    name: Lottery
    version: 1.0.0
  registry:
    address: nacos://127.0.0.1:8848
  #    address: N/A #multicast://224.5.6.7:1234 #测试用的是直连模式
  protocol:
    name: dubbo
    port: -1
  scan:      #扫描dubbo对外接口
    base-packages: cn.itedus.lottery.rpc.*

```

然后消费方使用

```java
@DubboReference(interfaceClass = 对外暴露的接口.class)
```

对于yml中的配置参考

[dubbo结合nacos](https://cn.dubbo.apache.org/zh/docs3-v2/java-sdk/reference-manual/registry/nacos/)


nacos还可以作为dubbo的配置中心