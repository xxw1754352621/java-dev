# 消费模式
  - 发布者生产一条消息到topic中，不同订阅组消费此消息
  - 点对点（只有一个消费者拿到）
  - [发布/订阅](https://cloud.tencent.com/developer/article/1116301)（跟其他消费组的消费者拿到的都是同样的数据）
  
# 消费状态和订阅关系维护

# rabbitmq和kafka的消费者
   - kafka
      - 消费端负责负载和幂等操作
        - 消费者负责记录偏移量，服务端一个分区挂了，重新负载，会出现重复消费
    
   - rabbitmq
      - 消费端只有msg_id
        - 服务端根据msg_id，找到索引，处理索引（前置），索引再找到磁盘的数据等操作（后置）

# kafka问题

- 消息丢失（服务端设置不合理）

  - 当分区的leader宕机，需要通过zookeeper实现leader选举，从新选出leader，但是，因为热备原因，备选的分区如果没有完全copy到leader分区的信息的情况下（如果确认模式是不等备份机copy确认），会造成消息丢失

- 消息重复（消费端不可用，重新分配消费端，rebalance）

  - 心跳检查（session.timeout.ms）
    - 消费端定期向服务端的coordinator发送心跳，检查消费端是否可用

  - 最大的拉取（poll）时间间隔（max.poll.interval.ms）
    - 一次拉取数量（max.poll.records）
    - 如果消费时间大于拉取间隔，也认为消费端不可用
      - 计算：拉取数量 * 每个消息的消费时间 > 拉取时间间隔 

  - 消费端宕机