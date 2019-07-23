# 理论

## [CAP](https://juejin.im/post/5c0e5bf8e51d45063322fe50#heading-24)

C->一致性，A->可用性，P->分区容错性

一致性(Consistency): 指在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）

可用性(Availability): 在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）

[分区容忍性](https://www.jianshu.com/p/2c30d1fe5c4e)(Partition tolerance): 即当节点之间无法正常通信时，就产生了分区，而分区产生后，依然能够保证服务可用，那么我们就说系统是分区容忍的。显然如果节点越多，且备份越多，分区容忍度就越高（因为即便是其中一个或多个节点挂了，仍然有其它节点和备份可用）。
 - 参考：https://www.hollischuang.com/archives/666
## CAP的延伸 - BASE

基本可用（Basically Available）、软状态（ Soft State）、最终一致性（ Eventual Consistency），核心思想是即使无法做到强一致性（**CAP 的一致性就是强一致性**），但应用可以采用适合的方式达到最终一致性。

