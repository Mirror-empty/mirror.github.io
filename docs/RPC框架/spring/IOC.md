# IOC

### Spring Bean 容器是什么？

Spring 包含并管理应用对象的配置和生命周期，在这个意义上它是一种用于承载对象的容器，你可以配置你的每个 Bean 对象是如何被创建的，这些 Bean 可以创建一个单独的实例或者每次需要时都生成一个新的实例，以及它们是如何相互关联构建和使用的。

一个Bena对象交给Spring 容器管理，然后把这个Bean对象拆解存放到Bean的定义中，相当于把对象解耦。

然后Bean对象被定义存放以后，再由Spring 统一进行装配，这个过程包括Bean的初始化、属性填充。所以就有一个词叫做自动装配。

能存放数据的数据结构称之为容器。

![pSM7QfO.png](https://s1.ax1x.com/2023/01/14/pSM7QfO.png)



### 实现 Bean 的定义、注册、获取

对Spring Bean 容器进行功能完善，实现 Bean 容器关于 Bean 对象的注册和获取。


基本流程：

```java
 @Test
    public void test_BeanFactory(){
        // 1.初始化 BeanFactory
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 2.注册 bean       这里涉及到自动装配，如果是属性自动装配，存入到map 中的key就是属性值
        BeanDefinition beanDefinition = new BeanDefinition(UserService.class);
        beanFactory.registerBeanDefinition("userService", beanDefinition);

        // 3.第一次获取 bean       
        UserService userService = (UserService) beanFactory.getBean("userService");
        userService.queryUserInfo();

        // 4.第二次获取 bean from Singleton
        UserService userService_singleton = (UserService) beanFactory.getSingleton("userService");
        userService_singleton.queryUserInfo();
    }

```
`` (UserService) beanFactory.getBean("userService");`` 
- 在获取bean中去实现了getSingleton，查看是否已经注册了单例对象，
- 没有的话根据String 去Bean定义类中获取定义值 交给createBean方法去创造bean对象
- 然后去createBean中创造一个bean实例对象，添加到单例注册表中。
- 只要注册过了，下次就能直接从单例注册表直接获取。


![pS1m5xf.png](https://s1.ax1x.com/2023/01/16/pS1m5xf.png)


### 对象实例化策略

