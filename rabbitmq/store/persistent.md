# 进程模型

- **tcp_acceptor**：负责接受客户端连接，然后为客户端连接创建rabbit_reader、rabbit_writer、rabbit_channel进程
  - 一个服务实例一个
- **rabbit_reader**：负责解析客户端AMQP帧，然后将请求发送给rabbit_channel进程
  - 一个客户端一个
- **rabbit_writer**：负责向客户端返回数据
  - 一个客户端一个
- **rabbit_channel**：负责解析AMQP方法，以及对消息进行路由，然后发送给对应的队列进程。
  - 一个TCP连接多个channel
- **rabbit_amqqueue_process**：rabbit队列进程，该进程一般在rabbitmq创建队列时被创建，其主要负责消息的接收/投递逻辑
  - 一个队列一个
- **rabbit_msg_store**：存储服务器进程，主要负责消息的持久化存储
  - 一个服务实例一个

# 消息存储过程

持久化两部分：队列索引（rabbit_queue_index）和消息存储(rabbit_msg_store)

队列索引：消息落盘的信息，包括消息的存储地点，是否交付给消费者，是否被消费者ack

消息存储：rabbit_msg_store以键值对的形式存储消息，每个节点有且只有一个，所有队列共享。从技术层面讲rabbit_msg_store又可以分为msg_store_persistent和msg_store_transient，其中msg_store_persistent负责持久化消息的存储，不会丢失，而msg_store_transient负责非持久化消息的存储，重启后消息会丢失。
 - Config file	/usr/local/rabbitmq_server-3.6.6/etc/rabbitmq/rabbitmq.config (not found)
 - 持久化mnesia ， Database directory	/usr/local/rabbitmq_server-3.6.6/var/lib/rabbitmq/mnesia/rabbit@localhost
 - Log file	/usr/local/rabbitmq_server-3.6.6/var/log/rabbitmq/rabbit@localhost.log
 - SASL log file	/usr/local/rabbitmq_server-3.6.6/var/log/rabbitmq/rabbit@localhost-sasl.log

### 1.消息入队

- 是否有消费者等待消费
  - 有，直接给消费者的channel进程
  - 没有，调用send_mandatory(Delivery)方法，判断mandatory
    - spring.rabbitmq.template.mandatory=false，没有路由到队列，丢弃
    - spring.rabbitmq.template.mandatory=true，没有路由到队列，给回生产的回调方法
  - 没有，是否重复BQ:is_duplicate
  - 没有，检查消息过期drop_expired_msgs
  - 只有持久化队列和消息持久化才会对消息进行持久化 
- 没有消费者，BQ:publish发布到BQ队列
  - 如果调用到该方法的BQ:publish则说明当前队列没有消费者正在等待，消息将进入到队列。
      backing_queue实现了消息的存储，他会尽力会durable=true的消息做持久化存储。
      初始默认情况下，非持久化消息直接进入内存队列，此时效率最高，
      当内存占用逐渐达到一个阈值时，消息和消息索引逐渐往磁盘中移动，
      随着消费者的不断消费，内存占用的减少，消息逐渐又从磁盘中被转到内存队列中。
      消息在这些Queue中传递的"一般"过程q1->q2->delta->q3->q4，
      一般负载较轻的情况消息不需要走完每个Queue，大部分都可以跳过
  - RabbitMQ系统中使用的内存过多，此操作是将内存中的队列数据写入到磁盘中， reduce_memory_use(maybe_update_rates(State3))，**一切有序**
   - 先判断q3是否为空
     - 为空，整个队列为空，队列进程直接给内存队列q4
     - **不为空，队列进程给q1**
   - （push_alphas_to_betas）内存消耗大（设置参数），alphas状态（q1，q4）转 deltas状态（q2，q3）
     - q1判断delta是否为空
       - 为空，存进q3 （小量）
       - 不为空，存进q2
     - q4存进q3（将Q4队列尾部的元素不断的放入到Q3队列的头部）
   - （push_betas_to_deltas）当前beta类型的消息大于允许的beta消息的最大值，则将beta类型多余的消息转化为deltas类型的消息
     - q2 + q3转移到delta 
   - 当内存紧张时触发paging，paging将大量alpha状态的消息转换为beta和gamma；如果内存依然紧张，继续将beta和gamma状态转换为delta状态。Paging是一个持续过程，涉及到大量消息的多种状态转换，所以Paging的开销较大，严重影响系统性能
### 2.消息消费
读取消息的时候先根据消息的msg_id找到对应的文件，如果文件存在且未被锁住则直接打开文件，如果文件不存在或者锁住了则发请求到rabbit_msg_store处理。
每个rabbit_queue_index从磁盘读取消息的时候至少读取一个段文件。

  - 参考：[rabbitmq消费端](https://github.com/xxw1754352621/java-dev/tree/master/rabbitmq/consumer/consume.md)
