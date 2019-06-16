# vm_memory_high_watermark
表示RabbitMQ使用内存的上限为系统内存的40%。也可以通过absolute参数制定具体可用的内存数。当RabbitMQ使用内存超过这个限制时，RabbitMQ 将对消息的发布者进行限流，直到内存占用回到正常值以内。
# vm_memory_high_watermark_paging_ratio
表示当RabbitMQ达到0.4*0.75=30%，系统将对queue中的内容启用paging机制，将message等内容换页到disk 中。
# queue_index_embed_msgs_below（4096 字节）
嵌入到索引的消息大小限制
# queue_index_max_journal_entries（32768 条）
队列的索引日志超过该阈值将刷新到磁盘
# queue_explicit_gc_run_operation_threshold	（1000条）
在内存压力下，正常队列设置的值，该值可以触发垃圾回收和减少内存使用，降低该值，会降低性能，提高该值，会导致更高的内存消耗
# lazy_queue_explicit_gc_run_operation_threshold（1000 条）
在内存压力下为延迟队列设置的值，该值可以触发垃圾回收和减少内存使用，降低该值，会降低性能，提高该值，会导致更高的内存消耗
#如果生产者投递快，建议使用延时队列，他也会减低内存的使用
https://www.rabbitmq.com/lazy-queues.html
