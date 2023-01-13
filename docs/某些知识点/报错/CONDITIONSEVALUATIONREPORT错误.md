# CONDITIONSEVALUATIONREPORT错误


这种错误少了@Service注解，遇到过两次了。

![pSCN94e.png](https://s1.ax1x.com/2023/01/01/pSCN94e.png)

报错主要还是看报错内容，自己去查看就行

```java

org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'cn.itedus.lottery.test.domain.ActivityTest': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'cn.itedus.lottery.domain.activity.service.deploy.IActivityDeploy' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@javax.annotation.Resource(shareable=true, lookup=, name=, description=, authenticationType=CONTAINER, type=class java.lang.Object, mappedName=)}
	
```