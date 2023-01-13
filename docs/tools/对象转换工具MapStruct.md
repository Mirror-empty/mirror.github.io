# 对象转换工具MapStruct

### @Mapping注解

@Mapper——表示该接口作为映射接口，编译时MapStruct处理器的入口

- uese：外部引入的转换类；
- componentModel：就是依赖注入，类似于在spring的servie层用@servie注入，那么在其他地方可以使用@Autowired取到值。该属性可取的值为
  - 默认：这个就是经常使用的 xxxMapper.INSTANCE.xxx;
  - cdi：使用该属性，则在其他地方可以使用@Inject取到值；
  - spring：使用该属性，则在其他地方可以使用@Autowired取到值；
  - d）jsr330/Singleton：使用者两个属性，可以再其他地方使用@Inject取到值;
- unmappedTargetPolicy: 它的所有方法都将忽略未映射的属性：

```java
@Mapper
public interface CarMapper {

    @Mapping(source = "make", target = "manufacturer")
    @Mapping(source = "numberOfSeats", target = "seatCount")
    CarDto carToCarDto(Car car);

    @Mapping(source = "name", target = "fullName")
    PersonDto personToPersonDto(Person person);
}
```

该@Mapper注释将使得MapStruct代码生成器创建的执行CarMapper过程中生成时的界面。

在生成的方法实现中，源类型（例如Car）中的所有可读属性将被复制到目标类型中的相应属性中（例如CarDto）：

- 当某个属性与其目标实体对应的名称相同时，它将被隐式映射。

- 当属性在目标实体中具有不同的名称时，可以通过@Mapping注释指定其名称。

### @Mapping

@Mapping——一对映射关系

- target：目标属性，赋值的过程是把“源属性”赋值给“目标属性”；

- source：源属性，赋值的过程是把“源属性”赋值给“目标属性”；

- dateFormat：用于源属性是Date，转化为String;

- numberFormat：用户数值类型与String类型之间的转化;

- constant：不管源属性，直接将“目标属性”置为常亮；

- expression：使用表达式进行属性之间的转化;

- ignore：忽略某个属性的赋值;

- qualifiedByName：根据自定义的方法进行赋值;

- defaultValue：默认值;


### 检索映射器

[检索映射器](http://www.kailing.pub/MapStruct1.3/index.html#mappers-factory)

**一般由工厂来声明**

```java
@Mapper
public interface CarMapper {

    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );

    CarDto carToCarDto(Car car);
}
```

### MapperConfig

共享配置，让其他接口来实现它

MapStruct提供了通过指向注释的中央接口来定义共享配置的可能性@MapperConfig。要使映射器使用共享配置，需要在@Mapper#config属性中定义配置界面。

该@MapperConfig注释具有相同的属性@Mapper注释。未通过via提供的任何属性@Mapper都将从共享配置继承。指定@Mapper的属性优先于通过引用的配置类指定的属性


### @InheritInverseConfiguration(name = "sourceToTarget")

表示方法继承相应的反向方法的反向配置

### @InheritConfiguration(name = "sourceToTarget")

指定映射方法