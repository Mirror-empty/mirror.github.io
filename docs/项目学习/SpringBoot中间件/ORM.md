# ORM

### ORM框架基本实现流程

- 获取xml文件，对xml文件进行解析，
- SqlSessionFactoryBuilder类（SqlSession 工厂） 构建实例化元素（参数映射、SQL解析、SQL执行、结果映射）
- 建立连接，然后调用相对应的接口方法（JDBC操作）
- 关闭sqlSession和reader。



### spring与ORM框架集合

##### 方案设计

在 Spring 中正常使用 DAO 时，是需要注入相应的类属性中，那么就需要把代理类的 Bean 注册到 Spring 容器中，交给 Spring 管理。

在方案设计中包括，对需要注册对象的扫描、代理类的实现、Bean的注册，这些是整个把 ORM 结合Spring 的核心内容。

当所有的内容实现以后，就可以通过 SqlSessionFactoryBuilder 连接到 ORM 框架。


### 实现

- MapperScannerConfigurer：去扫描classpath*：下的文件，然后注册资源

```java
public class MapperScannerConfigurer implements BeanDefinitionRegistryPostProcessor {

    private String basePackage;
    private SqlSessionFactory sqlSessionFactory;

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        try {
            // classpath*:cn/bugstack/**/dao/**/*.class     类的扫描注册
            String packageSearchPath = "classpath*:" + basePackage.replace('.', '/') + "/**/*.class";

            ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();

//            解析classpath*:下的所有文件 获取资源信息
            Resource[] resources = resourcePatternResolver.getResources(packageSearchPath);

//            遍历资源信息，注册到bean
            for (Resource resource : resources) {
                MetadataReader metadataReader = new SimpleMetadataReader(resource, ClassUtils.getDefaultClassLoader());

                ScannedGenericBeanDefinition beanDefinition = new ScannedGenericBeanDefinition(metadataReader);
                String beanName = Introspector.decapitalize(ClassUtils.getShortName(beanDefinition.getBeanClassName()));

                beanDefinition.setResource(resource);
                beanDefinition.setSource(resource);
                beanDefinition.setScope("singleton");
                beanDefinition.getConstructorArgumentValues().addGenericArgumentValue(beanDefinition.getBeanClassName());
                beanDefinition.getConstructorArgumentValues().addGenericArgumentValue(sqlSessionFactory);
//
                beanDefinition.setBeanClass(MapperFactoryBean.class);

                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(beanDefinition, beanName);
                registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```

- MapperFactoryBean：代理

```java

    /**
     * InvocationHandler 动态代理
     * @return
     * @throws Exception
     */
    @Override
    public T getObject() throws Exception {
//        Object proxy, Method method, Object[] args
        InvocationHandler handler = (proxy, method, args) -> {
            logger.info("你被代理了，执行SQL操作！{}", method.getName());
            if ("toString".equals(method.getName())) return null; // 排除Object方法
            try {
//                对SQL的操作进行分发处理
                return sqlSessionFactory.openSession().selectOne(mapperInterface.getName() + "." + method.getName(), args[0]);
            } catch (Exception e) {
                e.printStackTrace();
            }

            return method.getReturnType().newInstance();
        };
//        最终返回了执行结果，关于查询到结果信息会反射操作成对象类，这里就是我们实现的 ORM 中间件负责的事情了。
        return (T) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class[]{mapperInterface}, handler);
    }
```

- SqlSessionFactoryBean 

```java
/**
 * BeanFactory: 获取 Spring Bean的接口，也就是IOC容器。ApplicationContext 就是一个BeanFactor。
 * FactoryBean: T getObject()  获取泛型T的实例。用来创建Bean。
 * 当IoC容器通过getBean方法来FactoryBean创建的实例时实际获取的不是FactoryBean 本身而是具体创建的T泛型实例。
 * Class<?> getObjectType() 获取 T getObject()中的返回值 T 的具体类型
 * default boolean isSingleton() 用来规定 Factory创建的的bean是否是单例。
 * InitializingBean： 初始化bean的方式
 *
 * 帮我们处理mybaits 核心流程类的加载过程
 */
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean {
```

>BeanFactory与FactoryBean的区别

 * BeanFactory: 获取 Spring Bean的接口，也就是IOC容器。ApplicationContext 就是一个BeanFactor。
 * FactoryBean: T getObject()  获取泛型T的实例。用来创建Bean。
 * 当IoC容器通过getBean方法来FactoryBean创建的实例时实际获取的不是FactoryBean 本身而是具体创建的T泛型实例。
 * Class<?> getObjectType() 获取 T getObject()中的返回值 T 的具体类型
 * default boolean isSingleton() 用来规定 Factory创建的的bean是否是单例。
 * InitializingBean： 初始化bean的方式

### 结合springBoot 开发 ORM Starter

##### 方案设计

在 SpringBoot 的Starter 开发中，主要的核心在于自动加载 AutoConifguration 的使用，这里我们需要加载yml或者其他的配置文件，并结合配置信息扫描注册相关的 Bean 类信息，以及初始化相关的组件启动对象。

### 实现

整个中间件分为三块 autoconfigure、mybatis、spring，

- autoconfigure：读取自定义配置信息以及负责把相关 mybatis、spring 中的Bean 加载启动。
- mybatis：SqlSessionFactoryBuilder 做了符合 yml 配置方式的加载处理。
- spring： MapperScannerConfigurer 关于扫描定义 Bean 信息时 addGenericArgumentValue 入参信息的变更。


