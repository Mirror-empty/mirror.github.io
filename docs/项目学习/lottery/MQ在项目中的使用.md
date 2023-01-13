# MQ在项目中的使用

用的kafka

### 生产者的使用

声明主题，注入到bean中，发送方法返回ListenableFuture。

```java
    public ListenableFuture<SendResult<String,Object>> sendLotteryInvoice(InvoiceVO invoice){
        String objJson = JSON.toJSONString(invoice);
        logger.info("发送MQ消息 topic：{} bizId：{} message：{}", TOPIC_INVOICE, invoice.getuId(), objJson);
        return kafkaTemplate.send(TOPIC_INVOICE,objJson);
    }
```

消息发送成功或者失败在callback在流程中使用

### 消费者的使用

监听相对应的主题，对消息的处理捕获异常，交给work去使用

```java
    @KafkaListener(topics = "lottery_invoice",groupId = "lottery")
    public void onMessage(ConsumerRecord<?, ?> record, Acknowledgment ack, @Header(KafkaHeaders.RECEIVED_TOPIC) String topic) {
        Optional<?> message = Optional.ofNullable(record.value());
        // 1. 判断消息是否存在
        if (!message.isPresent()) {
            return;
        }
        // 2.处理 MQ 消息
        try {
            // 1.转化对象
            InvoiceVO invoiceVO = JSON.parseObject((String) message.get(), InvoiceVO.class);
            // 2.获取发送奖品工厂，执行发奖
            IDistributionGoods distributionGoodsService = distributionGoodsFactory.getDistributionGoodsService(invoiceVO.getAwardType());
            DistributionRes distributionRes = distributionGoodsService.doDistribution(new GoodsReq(invoiceVO.getuId(), invoiceVO.getOrderId(), invoiceVO.getAwardId(), invoiceVO.getAwardName(), invoiceVO.getAwardContent()));
            Assert.isTrue(Constants.AwardState.SUCCESS.getCode().equals(distributionRes.getCode()), distributionRes.getInfo());
            // 3. 打印日志
            logger.info("消费MQ消息，完成 topic：{} bizId：{} 发奖结果：{}", topic, invoiceVO.getuId(), JSON.toJSONString(distributionRes));
            // 4. 消息消费完成
            ack.acknowledge();
        }catch (Exception e) {
            // 发奖环节失败，消息重试。所有到环节，发货、更新库，都需要保证幂等。
            logger.error("消费MQ消息，失败 topic：{} message：{}", topic, message.get());
            throw e;
        }
```

### 在流程中添加

```java
 // 4. 发送MQ，触发发奖流程
        InvoiceVO invoiceVO = buildInvoiceVO(drawOrderVO);
        ListenableFuture<SendResult<String, Object>> future = kafkaProducer.sendLotteryInvoice(invoiceVO);
        future.addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {

            @Override
            public void onSuccess(SendResult<String, Object> result) {
                // 4.1 MQ 消息发送完成，更新数据库表 user_strategy_export.mq_state = 1
                activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.COMPLETE.getCode());
            }

            @Override
            public void onFailure(Throwable throwable) {
                // 4.2 MQ 消息发送失败，更新数据库表 user_strategy_export.mq_state = 2 【等待定时任务扫码补偿MQ消息】
                activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.FAIL.getCode());
            }

        });
```

对于一些报错 doPartake中 会去查询位置线抽奖领取了过程失败，可以直接返回领取结果继续抽奖，这里没弄take报null

还有

```java
Caused by: com.alibaba.fastjson.JSONException: syntax error, expect {, actual string, pos 0, fastjson-version 1.2.76
```

这个错误是由于转换对象的原因

这种错误的使用toJSONString会带有转义字符\

```java
    Object data = message.get();
    String string = JSON.toJSONString(data);
```

对的

```java
  InvoiceVO invoiceVO = JSON.parseObject((String) message.get(), InvoiceVO.class);
```