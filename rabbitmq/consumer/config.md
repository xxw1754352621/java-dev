# 消费者的确认模式（AcknowledgeMode）
  - 自动确认
    - 当服务端推送到消费端的时候，就认为消息投递成功，如果消费端在正常的处理过程中
     消费端被kill -9的话，就会丢失数据

  - 手动确认
     - 要等消费端完全消费之后，手动跟服务端确认消息已经消费完成，这样保证消息不丢失
       但是减低tps
# 预期消息数量（prefetch） 
  -  Maximum number of unacknowledged messages that can be outstanding 
    at each consumer.
  -  根据消费端的时候消息情况配置大小，消费能力越强，设置数量可以越大
  -  一般来讲，数据也大，消费端就也能帮服务端分压
# 投递模式delivery-mode 
  - 平衡速度和安全
# 消费者拒绝 basicNack(long deliveryTag, boolean multiple, boolean requeue)
   - deliveryTag
     - 发布的每一条消息都会获得一个唯一的deliveryTag（自增），deliveryTag在channel范围内是唯一的
   - multiple
     - 如果值为true，包含本条消息在内的、所有比该消息deliveryTag值小的 消息都被拒绝了（除了已经被 ack 的以外）;
     - 如果值为false，只拒绝三本条消息 