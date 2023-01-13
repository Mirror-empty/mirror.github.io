# redis在项目中的使用

添加一个redis的分布式锁，锁的需要细化颗粒度

```java
    /**
     * 分布式锁
     * @param key               锁住的key
     * @param lockExpireMils    锁住的时长。如果超时未解锁，视为加锁线程死亡，其他线程可夺取锁
     * @return
     */
    public boolean setNx(String key, Long lockExpireMils) {
        return (boolean) redisTemplate.execute((RedisCallback) connection -> {
            //获取锁
            return connection.setNX(key.getBytes(), String.valueOf(System.currentTimeMillis() + lockExpireMils + 1).getBytes());
        });
    }
```

看参数，一个是key，一个时间，key就是这个锁的键

![pSnAaRA.png](https://s1.ax1x.com/2023/01/11/pSnAaRA.png)

``del keynot`` 直接删掉，因为服务器可能会发生宕机，不能执行删除，因此就要去设置时长

使用redisTemplate的execute的回调方法，在里面使用setNX方法。RedisCallback类作用是进行回调

```java
    @Nullable
    public <T> T execute(RedisCallback<T> action, boolean exposeConnection) {
        return this.execute(action, exposeConnection, false);
    }
```

exposeConnection的布尔类型参数，那么这个参数是干什么的？有什么作用呢？

exposeConnection参数的含义是是否要暴露connection，如果为true，那么就可以在回调函数中使用当前连接connection对象。

返回一个布尔类型，来告诉分布式锁是否成功


```java
        // 2. 首次成功领取活动，发送 MQ 消息 为了库存和redis一致性？  用户如果已经领取过活动了
        //但是没有去抽奖 领取过活动代表 它库存已经删减过了
        if (Constants.ResponseCode.SUCCESS.getCode().equals(partakeResult.getCode())) {
            ActivityPartakeRecordVO activityPartakeRecord = new ActivityPartakeRecordVO();
            activityPartakeRecord.setuId(req.getuId());
            activityPartakeRecord.setActivityId(req.getActivityId());
            activityPartakeRecord.setStockCount(partakeResult.getStockCount());
            activityPartakeRecord.setStockSurplusCount(partakeResult.getStockSurplusCount());
            // 发送 MQ 消息
            kafkaProducer.sendLotteryActivityPartakeRecord(activityPartakeRecord);
        }
```

利用mq去做到redis和数据库的一致性，发送到mq中的对象的内容都是通过库存来查询的

```xml
    <update id="updateActivityStock" parameterType="cn.itedus.lottery.infrastructure.po.Activity">
        UPDATE activity SET stock_surplus_count = #{stockSurplusCount}
        WHERE activity_id = #{activityId} AND stock_surplus_count > #{stockSurplusCount}
    </update>
```

可以看下sql，这里去判断原先的库存要去大于扣减的库存才能进行数据库的更新，然而数据库原先的库存小于扣减的库存不进行更新，会有消息的堆积？



怎么去判断库存的

```java
        //  1. 获取抽奖活动库存 Key  
        String stockKey = Constants.RedisKey.KEY_LOTTERY_ACTIVITY_STOCK_COUNT(activityId);

        // 2. 扣减库存，目前占用库存数
        Integer stockUsedCount = (int) redisUtil.incr(stockKey, 1);
```

stockKey是根据活动id生成的字符串，固定的，然后每次去锁一次就相当于扣减一次库存。当然锁的key每次都是不同的，颗粒要更小，细化到库存的多少来作为key

```java
   // 4. 以活动库存占用编号，生成对应加锁Key，细化锁的颗粒度
        String stockTokenKey = Constants.RedisKey.KEY_LOTTERY_ACTIVITY_STOCK_COUNT_TOKEN(activityId, stockUsedCount);

        // 5. 使用 Redis.setNx 加一个分布式锁
        boolean lockToken = redisUtil.setNx(stockTokenKey, 350L);
        if (!lockToken) {
            logger.info("抽奖活动{}用户秒杀{}扣减库存，分布式锁失败：{}", activityId, uId, stockTokenKey);
            return new StockResult(Constants.ResponseCode.ERR_TOKEN.getCode(), Constants.ResponseCode.ERR_TOKEN.getInfo());
        }

```


```java

            // 3. 更新数据库库存【实际场景业务体量较大，可能也会由于MQ消费引起并发，对数据库产生压力，
            // 所以如果并发量较大，可以把库存记录缓存中，并使用定时任务进行处理缓存和数据库库存同步，减少对数据库的操作次数】
            activityPartake.updateActivityStock(activityPartakeRecordVO);
            // 4. 消息消费完成
            ack.acknowledge();
```

kafka消费重复问题，因为没有写消息消费完成的回调。

```java
ack.acknowledge();
```