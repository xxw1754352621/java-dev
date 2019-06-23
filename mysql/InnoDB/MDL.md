# Metadata lock([元数据锁](https://www.cnblogs.com/ivictor/p/9459265.html))

MDL是mysql5.5.3引入的表级锁
这是Server 层实现的锁，跟引擎层无关  
当你执行select的时候，如果这时候有ddl语句，那么ddl会被阻塞，因为select语句拥有metadata lock，防止元数据被改掉


- 没有MDL，带来的问题

  - RR隔离级别下，依然出现不可重复读的问题
    - 当前事务，在DDL后查询，跟DDL前查询的结果不一样（不正确），
  - 主从数据不一致的问题，可以查看binlog日志对比
    - 同一个事务的两个sql（一个DML，一个是DDL）在另外一台机器的binlog日志执行顺序不一样
  - 主从复制中断/阻塞问题

- MDL可能带来的问题

  - DDL操作获取DML锁，其他对该表的操作，都会阻塞。阻塞稍久的话，我们会看到Threads_running飙升，CPU告警
    
    - 提示waiting for table metadata lock
    
    - MDL_EXCLUSIVE是独占锁，在其持有期间是不允许**其它类型的MDL**被授予，自然也包括SELECT和DML操作。
    
      这也就是为什么DDL操作被阻塞时，后续其它操作也会被阻塞。

 ```properties
1. MDL出现的初衷就是为了保护一个处于事务中的表的结构不被修改。

2. 这里提到的事务包括两类，显式事务和AC-NL-RO（auto-commit non-locking read-only）事务。显式事务包括两类：1. 关闭AutoCommit下的操作，2. 以begin或start transaction开始的操作。AC-NL-RO可理解为AutoCommit开启下的select操作。

3. MDL是事务级别的，只有在事务结束后才会释放。在此之前，其实也有类似的保护机制，只不过是语句级别的。
 ```

​    

- 需要注意的是，MDL不仅仅适用于表，同样也适用于其它对象，如下表所示，其中，"等待状态"对应的是"show processlist"中的State。

```properties
为了提高数据库的并发度，MDL被细分为了11种类型。

MDL_INTENTION_EXCLUSIVE

MDL_SHARED

MDL_SHARED_HIGH_PRIO

MDL_SHARED_READ

MDL_SHARED_WRITE

MDL_SHARED_WRITE_LOW_PRIO

MDL_SHARED_UPGRADABLE

MDL_SHARED_READ_ONLY

MDL_SHARED_NO_WRITE

MDL_SHARED_NO_READ_WRITE

MDL_EXCLUSIVE

常用的有MDL_SHARED_READ,MDL_SHARE D_WRITE及MDL_EXCLUSIVE，其分别用于SELECT操作，DML操作及DDL操作。其它类型的对应操作可参考源码sql/mdl.h。
```

