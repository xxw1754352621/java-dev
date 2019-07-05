# mybatis缓存

缓存目的

- 对于查询频繁的压力分担到客户端，减轻服务端的压力

缓存问题

- 数据一致性

缓存颗粒度

- Session会话级别
  - 一级缓存
- 应用级别
  - 二级缓存
    - 引入第三方缓存（EHcache，redis，Memcached）

缓存存储

- 内存-RAM
- 磁盘-DISK

回收策略（decorators包下）

```sql
1.FIFOCache：先进先出算法 回收策略，装饰类，内部维护了一个队列，来保证FIFO，一旦超出指定的大小，
则从队列中获取Key并从被包装的Cache中移除该键值对。
2.LoggingCache：输出缓存命中的日志信息,如果开启了DEBUG模式，则会输出命中率日志。
3.LruCache：最近最少使用算法，缓存回收策略,在内部保存一个LinkedHashMap
4.ScheduledCache：定时清空Cache，但是并没有开始一个定时任务，而是在使用Cache的时候，才去检查时间是否到了。
5.SerializedCache：序列化功能，将值序列化后存到缓存中。该功能用于缓存返回一份实例的Copy，用于保存线程安全。
6.SoftCache：基于软引用实现的缓存管理策略,软引用回收策略，软引用只有当内存不足时才会被垃圾收集器回收
7.SynchronizedCache：同步的缓存装饰器，用于防止多线程并发访问
8.PerpetualCache 永久缓存，一旦存入就一直保持，内部就是一个HashMap
9.WeakCache：基于弱引用实现的缓存管理策略
10.TransactionalCache 事务缓存，一次性存入多个缓存，移除多个缓存
11.BlockingCache 可阻塞的缓存,内部实现是ConcurrentHashMap
```

