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