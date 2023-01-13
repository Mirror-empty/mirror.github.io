# 策略模式的应用ID生成

不同的场景需要不同的ID，这里生成三种ID

- 订单号：唯一、大量、订单场景时使用、分库分表
- 活动号：唯一、少量、活动创建时使用、单库单表
- 策略好：唯一、少量、活动创建时使用、单库单表


![pSCD7lD.png](https://s1.ax1x.com/2023/01/01/pSCD7lD.png)

IIdGenerator接口为一个算法族，实现nextInt方法。

IdContext中的idGenerator实现Id的调用。

![pSCrFmj.png](https://s1.ax1x.com/2023/01/01/pSCrFmj.png)


SnowFlake雪花Id

```java
@Component
public class SnowFlake implements IIdGenerator {

    private Snowflake snowflake;

    @PostConstruct
    public void init(){
        long workerId;

        try {
//            根据long值获取ip v4地址   获取本机网卡IP地址，
            workerId = NetUtil.ipv4ToLong(NetUtil.getLocalhostStr());
        }catch (Exception e){
//            此方法不会抛出异常，获取失败将返回null
            workerId = NetUtil.getLocalhostStr().hashCode();
        }

//        保留32位
        workerId = workerId >> 16 & 31;

        long dataCenterId = 1L;
//          参数1为终端ID，参数2位数据中心ID
        snowflake = IdUtil.createSnowflake(workerId,dataCenterId);
    }


    @Override
    public synchronized long nextId() {
        return snowflake.nextId();

    }
}
```

时间日期id

```java
@Component
public class ShortCode implements IIdGenerator {
    @Override
    public synchronized long nextId() {
        Calendar calendar = Calendar.getInstance();
        int year = calendar.get(Calendar.YEAR);
        int week = calendar.get(Calendar.WEEK_OF_YEAR);
        int day = calendar.get(Calendar.DAY_OF_WEEK);
        int hour = calendar.get(Calendar.HOUR_OF_DAY);

//        打乱排序：2020年为准 + 小时 + 周期 + 日 +三位随机数
        StringBuilder idStr = new StringBuilder();
        idStr.append(year-2020);
        idStr.append(hour);
        idStr.append(String.format("%02d",week));
        idStr.append(day);
        idStr.append(String.format("%03d",new Random().nextInt(1000)));

        return Long.parseLong(idStr.toString());

    }
}
```

随机生成id用的是commons.lang3.RandomStringUtils

```java
@Component
public class RandomNumeric implements IIdGenerator {


    @Override
    public long nextId() {
        return Long.parseLong(RandomStringUtils.randomNumeric(11));
    }

}

```

重点对外开放的IDContext

```java

@Configuration
public class IdContext {
    /**
     * 创建 ID 生成策略对象，属于策略设计模式的使用方式
     *
     * @param snowFlake 雪花算法，长码，大量
     * @param shortCode 日期算法，短码，少量，全局唯一需要自己保证
     * @param randomNumeric 随机算法，短码，大量，全局唯一需要自己保证
     * @return IIdGenerator 实现类
     */
//    为什么要加bean呢？注入到容器里？
    @Bean
    public Map<Constants.Ids,IIdGenerator> idGenerator(SnowFlake snowFlake, ShortCode shortCode,
                                                       RandomNumeric randomNumeric){
        Map<Constants.Ids,IIdGenerator> idGeneratorMap = new HashMap<>(8);
        idGeneratorMap.put(Constants.Ids.SnowFlake,snowFlake);
        idGeneratorMap.put(Constants.Ids.ShortCode, shortCode);
        idGeneratorMap.put(Constants.Ids.RandomNumeric, randomNumeric);
        return idGeneratorMap;
    }
}

```

生成一个bean，注册到spring容器中，外部直接``@Resource Map<Constants.Ids,IIdGenerator> ``变量名就行


