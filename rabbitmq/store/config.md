# Memory-based Flow Control threshold（流控启动）(内存和磁盘使用量设置阈值)
### vm_memory_high_watermark
  - 表示RabbitMQ使用内存的上限为系统内存的40%。也可以通过absolute参数制定具体可用的内存数。当RabbitMQ使用内存超过这个限制时，RabbitMQ 将对消息的发布者进行限流，直到内存占用回到正常值以内。
### vm_memory_high_watermark_paging_ratio
  - 表示当RabbitMQ达到0.4*0.75=30%，系统将对queue中的内容启用paging机制，将message等内容换页到disk 中。
### [disk_free_limit（50M）](https://jusene.github.io/2018/03/05/rabbitmq-4/)
  - 存储数据分区的可用磁盘空间限制
### credit_flow_default_credit {400, 200}{InitialCredit，MoreCreditAfter},3.5.5版本之后
  - rabbitmq[流量控制](https://blog.csdn.net/vipshop_fin_dev/article/details/81612935),一种基于信用证的流控机制
  -  By default, every connection, channel or queue is given 400 credits,
     and then 200 for every 200 messages that it sends to a peer process.
     Increasing these values may help with throughput but also can be dangerous:
     high credit flow values are no different from not having flow control at all.

# 内存换出和垃圾回收
### queue_index_embed_msgs_below（4096 字节）
  - 嵌入到索引的消息大小限制
### queue_index_max_journal_entries（32768 条）
  - 队列的索引日志超过该阈值将刷新到磁盘
  - journal文件是queue_index为避免过多磁盘寻址添加的一层缓冲（内存文件）。
    对于生产消费正常的情况，消息生产和消费的记录在journal文件中一致，则不用再保存；对于无消费者情况，该文件增加了一次多余的IO操作。
### queue_explicit_gc_run_operation_threshold	（1000条）
  - 在内存压力下，正常队列设置的值，该值可以触发垃圾回收和减少内存使用，降低该值，会降低性能，提高该值，会导致更高的内存消耗
### lazy_queue_explicit_gc_run_operation_threshold（1000 条）
  - 在内存压力下为延迟队列设置的值，该值可以触发垃圾回收和减少内存使用，降低该值，会降低性能，提高该值，会导致更高的内存消耗
### 如果生产者投递快，建议使用[延时队列](https://www.rabbitmq.com/lazy-queues.html)，他也会减低内存的使用
