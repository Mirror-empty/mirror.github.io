# xxl-job任务调度在项目中的使用

执行器注册到xxljob需要一定的时间才能生效，默认30s，类似服务注册发现需要时间，还有内部也有心跳检测和服务端剔除操作

### xxl-job的使用

(参考官网)[https://www.xuxueli.com/xxl-job/#%E4%B8%80%E3%80%81%E7%AE%80%E4%BB%8B]

1. 导入sql文件，改admin下的application配置
2. 修改logback.xml文件中的日志打印路径

```java
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false" scan="true" scanPeriod="1 seconds">

    <contextName>logback</contextName>
    <property name="log.path" value="D:/xxl-job/jobhandler/data/applogs/xxl-job/xxl-job-admin.log"/>

    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
```

3. 启动执行器springboot ，一样修改配置文件中和logback中的打印日志路径

![pSZbyMq.png](https://s1.ax1x.com/2023/01/09/pSZbyMq.png)

任务管理对应着``    @XxlJob("demoJobHandler")`` 可以在编辑中去设置隔多少时间定时调度。

参考：
- [xxl-job快速入门](https://juejin.cn/post/6923508824758288398)
- [xxl-job快速入门](https://blog.csdn.net/xhmico/article/details/122324950)



### xxl-job应用在项目中

```java
 @Bean
    public XxlJobSpringExecutor xxlJobExecutor(){
        logger.info(">>>>>>>>>>> xxl-job config init.");

        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);

        return xxlJobSpringExecutor;

    }
```

将XxlJobSpringExecutor注入到bean中去，匹配Spring框架

```java
public class XxlJobSpringExecutor extends XxlJobExecutor implements ApplicationContextAware, SmartInitializingSingleton, DisposableBean {
}
```

- 实现ApplicationContextAware 是为了获取上下文。
- 实现DisposableBean是为了重写destroy方法，用于释放一些资源。
- 实现SmartInitializingSingleton，重点，在Bean创建的生命周期里面，单例bean都初始化完成以后，找出带有注解的job进行注册一系列操作。

```java
    public void afterSingletonsInstantiated() {
        this.initJobHandlerMethodRepository(applicationContext);
        GlueFactory.refreshInstance(1);

        try {
            super.start();
        } catch (Exception var2) {
            throw new RuntimeException(var2);
        }
    }
```

1. 初始化JobHandler，就是找到带有注解@XxlJob的Job，进行注册
2. 刷新GlueFactory, 这里type 为 0/1 ，1就是说明使用的是Spring 框架

[参考xxl-job源码分析](https://blog.csdn.net/mamamalululu00000000/article/details/115671024)


**执行任务管理**

```java
@XxlJob("lotteryActivityStateJobHandler")
    public void lotteryActivityStateJobHandler() throws Exception {
        logger.info("扫描活动状态 Begin");

        List<ActivityVO> activityVOList = activityDeploy.scanToDoActivityList(0L);
        if (activityVOList.isEmpty()){
            logger.info("扫描活动状态 End 暂无符合需要扫描的活动列表");
            return;
        }

            // 获取集合中最后一条记录，继续扫描后面10条记录
            ActivityVO activityVO = activityVOList.get(activityVOList.size() - 1);
            activityVOList = activityDeploy.scanToDoActivityList(activityVO.getId());
        }

        logger.info("扫描活动状态 End");

    }
```

需要进行的任务管理，@XxlJob的属性与运行模式相匹配。可以自动导入到admin中。