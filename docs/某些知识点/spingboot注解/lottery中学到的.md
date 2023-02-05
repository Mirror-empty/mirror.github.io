# lottery中学到的

### @Configuration

官方文档描述：

　　用@Conﬁguration注释类表明其主要目的是作为bean定义的源

　　@Conﬁguration类允许通过调用同一类中的其他@Bean方法来定义bean之间的依赖关系

没有配置类就没有Bean

表名当前类是一个配置类。

### @EnableDubbo

@EnableDubbo=@DubboComponentScan+@EnableDubboConfig

@DubboComponentScan是对注解 @Service和@Referce的处理 (主要是将这俩货交给spring容器)，

@EnableDubboConfig则是开启读取配置文件，读取我们自定义的配置信息。



### @Service

dubbo下的注解

添加了@Service注解的类。最后得到BeanDefinitionHolder对象，调用registerServiceBean来注册ServiceBean

buildServiceBeanDefinition这个方法用来构建ServiceBean对象，每个暴露出的服务，最终都会构建成一个ServiceBean,

最终dubbo解析出了ServiceBean对象的beandefinition放入了spring的容器。ServiceBean对于每个暴露的服务来说很重要，每个暴露的服务都拥有一个ServiceBean，这个类里面包含了服务发布，服务下线等操作，算是一个很核心的类，

暴露服务作用

在dubbo 2.7.8中注解
@Service被@DubboService 取代。
@Reference被@DubboReference取代。

估计是dubbo的开发团队考虑到，原来的注解和spring的原生注解重名了，为了在语言层面和spring的原生注解，有所以区别减少出错概率


### @Reference

在需要调用的服务接口上使用即可直接调用远程服务。
@Reference(version = "1.0.0",application = "${dubbo.application.id}")

参考
```java
    @Reference(interfaceClass = IActivityBooth.class, url = "dubbo://127.0.0.1:20880")
    private IActivityBooth activityBooth;
```


### @PostConstruct

用于在依赖关系注入完成之后需要执行的方法上，以执行任何初始化。

应用 PostConstruct 注释的方法必须遵守以下所有标准：该方法不得有任何参数

加载Servlet的时候运行，并且只会被服务器调用一次，类似于Serclet的inti()方法。被@PostConstruct修饰的方法会在构造函数之后，init()方法之前运行。

简单的说就是启动的时候就会执行此注解下的方法。

### @Transactional

声明注解下的东西，触发了相对应的异常就进行事务回滚


@Transactional注解可以作用于接口、接口方法、类以及类方法上
1. 当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性
2. 当作用在方法级别时会覆盖类级别的定义
3. 当作用在接口和接口方法时则只有在使用基于接口的代理时它才会生效，也就是JDK动态代理，而不是Cglib代理
4. 当在 protected、private 或者默认可见性的方法上使用 @Transactional 注解时是不会生效的，也不会抛出任何异常
5. 默认情况下，只有来自外部的方法调用才会被AOP代理捕获，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰

rollbackFor
该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如：
1. 指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)
2. 指定多个异常类：@Transactional(rollbackFor={RuntimeException.class, BusnessException.class})

就是对一种事务失败时进行的回滚，事务就是数据库中的一种操作。

[参考](https://cloud.tencent.com/developer/article/1495739)


### @Configuration注解

@Configuration标注在类上，相当于把类作为spring的xml配置文件中的，作用为：配置spring容器（应用上下文）

@COnfiguration + @Bean 注册bean对象 所以外部就可以直接应用了。


### @Target注解 @Documented注解 @Retention注解

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
```

**@Target**

指明了修饰的这个注解的使用范围，意思是这个注解可以用在哪里

ElmentType的取值：
- TYPE：类，接口或者枚举
- FLELD：域，不会枚举常量
- METHOD：方法
- parameter：参数
- constructor：构造方法
- Local_VARIABLE：局部变量
- ANNOTATION_TYPE：注解类型
- PACKAGE：包


**@Retention**

表示修饰的注解的生存周期，会保留到哪个阶段。

RetentionPolicy的取值包含以下三种：
- SOURCE：源码级别保留，编译后即丢弃
- CLASS：编译级别保留，编译后的class文件中汇总，在jvm运行时丢弃，这是默认值
- RUNTIME：运行级别保留，编译后的class文件中存在，在jvm运行时保留，可以被反射调用。


**@Documented**

指明修饰的注解，可以被例如javadoc此类的工具文档化，只负责标记，没有成员取值。


### @Intercepts

注解类，其value为Signature类的数值，注解在Interceptor实现类上，表示实现类对哪些sql执行类（实现Executor）的哪些方法切入

和@Signature配套使用

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({})
public @interface Signature {
  /**
   * 定义拦截的类 Executor、ParameterHandler、StatementHandler、ResultSetHandler当中的一个
   */
  Class<?> type();

  /**
   * 在定义拦截类的基础之上，在定义拦截的方法
   */
  String method();

  /**
   * 在定义拦截方法的基础之上在定义拦截的方法对应的参数，
   * JAVA里面方法可能重载，故注意参数的类型和顺序
   */
  Class<?>[] args();
}
```


### @RunWith和@SpringbootTest

@RunWith就是一个运行器

@RunWith(JUnit4.class)就是指用JUnit4来运行

–@RunWith(SpringJUnit4ClassRunner.class)，让测试运行于Spring测试环 境，以便在测试开始的时候自动创建Spring的应用上下文

@RunWith(Suite.class)的话就是一套测试集合

@RunWith是Junit4提供的注解，将Spring和Junit链接了起来。假如使用Junit5，不再需要使用@ExtendWith注解，@SpringBootTest和其它@*Test默认已经包含了该注解。

@SpringBootTest替代了spring-test中的@ContextConfiguration注解，目的是加载ApplicationContext，启动spring容器。

@SpringBootTest 自动侦测并加载@SpringBootApplication或@SpringBootConfiguration中的配置，默认web环境为MOCK，不监听任务端口


### @ConditionalOnMissingBean

@ConditionalOnMissingBean，它是修饰bean的一个注解，主要实现的是，当你的bean被注册之后，如果而注册相同类型的bean，就不会成功，它会保证你的bean只有一个，即你的实例只有一个，当你注册多个相同的bean时，会出现异常，以此来告诉开发人员。


### @ConditionalOnClass

作用是当项目中存在某个类时才会使标有该注解的类或方法生效；


### @EnableConfigurationProperties

@EnableConfigurationProperties 注解的作用是:让使用了 @ConfigurationProperties 注解的类生效,并且将该类注入到 IOC 容器中,交由 IOC 容器进行管理

### @Aspect

把被修饰的类定义⼀个切⾯，作⽤是把被注释修饰的类当成切⾯给容器读取

### @Pointcut

把被修饰的⽅法定义为⼀个切点，每个Pointcut的定义包括2部分，⼀是表达式，⼆是⽅法
签名。⽅法签名必须是 public及void型。可以将Pointcut中的⽅法看作是⼀个被Advice引⽤的助记符，因
为表达式不直观，因此我们可以通过⽅法签名的⽅式为 此表达式命名。因此Pointcut中的⽅法只需要⽅
法签名，⽽不需要在⽅法体内编写实际代码。对应的我们可以知道有⼗种表达式标签和组成的⼗⼆种⽤
法（https://zhuanlan.zhihu.com/p/153317556）

### 增加通知

- @Around：环绕增强，这⾥注意它并不是指的是执⾏两次，
- @AfterReturning：后置增强，相当于AfterReturningAdvice，⽅法正常退出时执⾏
- @Before：标识⼀个前置增强⽅法，相当于BeforeAdvice的功能，相似功能的还有
- @AfterThrowing：异常抛出增强，相当于ThrowsAdvice
- @After: final增强，不管是抛出异常或者正常退出都会执行

不看不知道感谢21打卡

其他的都好理解，这个@Around要注意它并不是字⾯意思切点前执⾏⼀次和切点后执⾏⼀次，我们来看看官⽅的解释：

```java
Around advice
runs "around" a matched method execution. It has the opportunity to
do work both before and after the method executes, and to determine
when, how, and even if, the method actually gets to execute at all.
Around advice is often used if you need to share state before and
after a method execution in a thread-safe manner (starting and
stopping a timer for example).
```

⼤概的意思是：

它有机会是否在⽅法执⾏之前和之后都⼯作，并确定
该⽅法何时、如何以及是否实际执⾏。如果您需要在和之前共享状态，则经常使⽤Around建议
⽅法以线程安全的⽅式执⾏(启动和例如停⽌计时器)。

就是说它只会执⾏⼀次，它是为了保证切点执⾏前后能保证在共享的状态⽽产⽣的。这是官⽅设计这个环绕通知的初衷，它能
保证线程安全的⽅式执⾏。

总的来说就是只会执行一次，去保证切点执行前后一样的执行？

保证线程安全的方式执行。

