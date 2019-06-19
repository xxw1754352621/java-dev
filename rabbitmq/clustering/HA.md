# 高可用队列（[HA](https://www.rabbitmq.com/ha.html)）

  - rabbitmqctl set_policy [-p Vhost] Name Pattern Definition [Priority]

```properties
-p Vhost： 可选参数，针对指定vhost下的queue进行设置
Name: policy的名称
Pattern: queue的匹配模式(正则表达式)
Definition：镜像定义，包括三个部分ha-mode, ha-params, ha-sync-mode
	ha-mode:指明镜像队列的模式，有效值为 all/exactly/nodes
		all：表示在集群中所有的节点上进行镜像
		exactly：表示在指定个数的节点上进行镜像，节点的个数由ha-params指定
		nodes：表示在指定的节点上进行镜像，节点名称通过ha-params指定
	ha-params：ha-mode模式需要用到的参数
	ha-sync-mode：进行队列中消息的同步方式，有效值为automatic和manual
priority：可选参数，policy的优先级
```

# 镜像交换机的意义
 - 因为交换机，队列，路由键等元信息，在集群任何一个节点都有
   - 交换器等 不会跟节点挂钩
   - 镜像交换机跟普通交换机没有明显的区别
   - 待挖掘，发送消息
# [镜像队列](https://blog.csdn.net/jaredcoding/article/details/78112016)
   - queue-master-locator策略
     - 普通集群下，持久化数据在主节点
     - 队列进程和内容维护在单个节点上
   - 镜像队列
     - 镜像到的节点，都会持久化，已保证主节点不可用的情况下，mirror节点升级为主节点
   - 高可用，数据不丢失
     - 没有新节点加入，队列都处已同步的状态下
   - master节点宕机的问题：
     - 数据丢失
       - 都是新节点加入，且都没有同步完成
         - ha-prornote-on-shutdown 设置为 when-synced，为了保证数据不丢失，其他slave队列不会接管master。
         - ha-prornote-on-shutdown 设置为 always，无论如何，slave队列接管master，**信息丢失**，保证了高可用
     - 信息重复（broker接收数据丢失TCP）
       - 新的 master 重新入队所有 **unack** 的消息，因为新的 lave 无法区分这些 unack 的消息是
         否己经到达客户端 ，或者是 ack 信息丢失在老的 master 链路上，再或者是丢失在老的 master
         ack 消息到所有 slave 的链路上，所以出于消息可靠性的考虑，重新入队所有 unack 的消息，
         不过此时客户端可能会有重复消息。
     - 消费者感知master宕机
       - 如果客户端连接着 slave ，并且 Basic.Consume 消费时指定了 x-cancel-on-ha-failover 参数，那么断开之时客户端会收到一个 Consumer Cancellation Notification 的通知，
         消费者客户端中会回调 Consumer 接口的 handleCancel 方法
#镜像队列的实现
  - 普通的 backing_queue
    - rabbit_variable_queue实现持久化
  - 镜像的 backing_queue    
    - rabbit_mirror_queue_master
    - rabbit_mirror_queue_slave
  - GM
    - 过组播 GM(Guaranteed Multicast) 的方式同步到各个 slave
  - coordinator
    - master 上的回调处理
    - 闭环，最终操作命令会回到master
# 目的
   - 数据冗余备份，实现高可用HA
   - 避免单点故障