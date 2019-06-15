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
 