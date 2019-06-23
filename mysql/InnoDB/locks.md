[TOC]

# 一次封锁和二段锁（[加锁和解锁](https://tech.meituan.com/2014/08/20/innodb-lock.html)）

一段锁可以避免死锁，mysql不采用这个的原因是，**事务**开始的时候，不知道要锁住那些数据

二段锁会出现死锁，衍生出MVCC，锁兼容等

| 事务                 | 加锁/解锁处理                                      |
| :------------------- | :------------------------------------------------- |
| begin；              |                                                    |
| insert into test ….. | 加insert对应的锁                                   |
| update test set…     | 加update对应的锁                                   |
| delete from test ….  | 加delete对应的锁                                   |
| commit;              | 事务提交时，同时释放insert、update、delete对应的锁 |

```properties
如果一个条件无法通过索引快速过滤，存储引擎层面就会将所有记录加锁后返回，再由MySQL Server层进行过滤。

但在实际使用过程当中，MySQL做了一些改进，在MySQL Server过滤条件，发现不满足后，会调用unlock_row方法，把不满足条件的记录释放锁 (违背了二段锁协议的约束)。这样做，保证了最后只会持有满足条件记录上的锁，但是每条记录的加锁操作还是不能省略的。可见即使是MySQL，为了效率也是会违反规范的。（参见《高性能MySQL》中文第三版p181）
```



# shared and Exclusive locks

读写锁（S 和 X）

- two types of row-level locking

  - 行锁的两种类型

- A request by `T2` for an `S` lock can be granted immediately（立即授权，批准）. As a result, both `T1` and `T2` hold an `S` lock on `r`.

  - 当前行拥有多个S锁

- A request by `T2` for an `X` lock cannot be granted immediately.

  - T2事务等待T1释放`X`锁
- T2事务同时向`r`不能加`S`锁
- 事务隔离级别serialable，才加读锁，不然就要显式加锁（for update等）

| 读写锁（T1和T2事务） | X                        | IX                 | S                  | IS                 |
| -------------------- | ------------------------ | ------------------ | ------------------ | ------------------ |
| X                    | ✖️​                        | ✖️                  | ✖️                  | ✖️                  |
| IX                   | :heavy_multiplication_x: | :heavy_check_mark: | ✖️                  | :heavy_check_mark: |
| S                    | ✖️                        | ✖️                  | :heavy_check_mark: | :heavy_check_mark: |
| IS                   | ✖️                        | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |



# [intention locks](https://juejin.im/post/5b85124f5188253010326360)

```properties
1.IX，IS是表级锁，不会和行级的X，S锁发生冲突。只会和**表级**的X，S发生冲突

2.意向锁（IS 和 IX），互相**不排斥**，在为数据行加共享 / 排他锁之前添加

3.目的：提高效率，避免T2获取表级锁的时候，要逐行检查是否有锁

4.目的：意向锁在保证并发性的前提下，实现了`行锁和表锁共存`且`满足事务隔离性`的要求
```



- InnoDB 支持`多粒度锁（multiple granularity locking）`，它允许`行级锁`与`表级锁`共存，而**意向锁**就是其中的一种`表锁`。

- An [intention shared lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_intention_shared_lock) (`IS`) indicates that a transaction intends to set a *shared* lock on individual rows in a table.
  - 一个事务打算往（个别）多行记录加读锁，获取读锁后，意向锁不会释放
- An [intention exclusive lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_intention_exclusive_lock) (`IX`) indicates that a transaction intends to set an exclusive lock on **individual rows** in a table.
  - 一个事务打算往（个别）多行记录加写锁，获取写锁后，意向锁依然持有
- For example, [`SELECT ... FOR SHARE`](https://dev.mysql.com/doc/refman/8.0/en/select.html) sets an `IS` lock, and [`SELECT ... FOR UPDATE`](https://dev.mysql.com/doc/refman/8.0/en/select.html) sets an `IX` lock.
  - 加锁的例子如上，可能只影响**一条记录**
- The intention locking protocol is as follows:
  - Before a transaction can acquire a shared lock on **a row** in a table, it must first acquire an `IS` lock or stronger on the table.
    - 在往一条记录加读锁的时候，先加意向读锁（`IS`）
    - 事务要获取某些行的 S 锁，必须先获得表的 IS 锁。
      **SELECT** **column** **FROM** **table** ... **LOCK** **IN** **SHARE** **MODE**;
  - Before a transaction can acquire an exclusive lock on a row in a table, it must first acquire an `IX` lock on the table.
    - 在往一条记录加写锁的时候，先加意向写锁（`IX`）
    - 事务要获取某些行的 X 锁，必须先获得表的 IX 锁。
      **SELECT** **column** **FROM** **table** ... **FOR** **UPDATE**;

# record locks

- 行锁（读写锁）
- Record locks always lock index records, even if a table is defined with no indexes. For such cases, `InnoDB` creates a hidden clustered index and uses this index for record locking. See [Section 15.6.2.1, “Clustered and Secondary Indexes”](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html).
  - 只会锁带索引的记录，即使没有显式添加索引，innodb也会创建一个隐式的聚集索引，锁住这个隐式索引的行记录
- the row-level locks are actually index-record locks
  - 行级锁实际上是索引记录锁
  - Innodb 中的`行锁`的实现依赖于`索引`，一旦某个加锁操作没有使用到索引，那么该锁就会退化为`表锁`。

# gap locks

A gap lock is a lock on a gap between index records, or a lock on the gap before the first or after the last index record

- 间隙锁，索引值之间的间隙

# next-key locks

A next-key lock is a combination of a record lock on the index record and a gap lock on the gap before the index record.

- 行锁防止别的事务修改或删除，GAP锁防止别的事务新增，行锁和GAP锁结合形成的的Next-Key锁共同解决了RR级别在**写数据**（`X锁`）时的幻读问题。

- 分析，条件如下：mysql，innodb，RR，test_table（primary id，key user_id，user_name）

  1. 当前行记录有user_id={3,19,30}，语句：

     ```mysql
     因为临键锁，我们先根据二级索引user_id，设定有以下数据分区
     (negative infinity, 3],
     (3,19],
     (19,30],
     (30,positive infinity);
     ```

  2.  update test_table set user_name="kaka" where user_id=19

     - 命中二级索引值19，所以添加临键锁
       - 行锁，锁住19
       - 间隙锁，锁住(3,19]和(19,30]

  3.  update test_table set user_name="kaka" where user_id=20

     - 没有命中行，降级为间隙锁
       - 没有行锁
       - 间隙锁，锁住(19,30]

  4.  update test_table set user_name="kaka" where id=19

     - 命中主键（或者唯一键（null值特殊）），降级为行锁
       - 行锁
       - 没有间隙锁，没有必要，因为唯一，无论如何，都不会影响这条sql的执行结果

# insert intention locks

- is a type of gap lock set by [`INSERT`](https://dev.mysql.com/doc/refman/8.0/en/insert.html) operations
  - 插入意向锁，**一种类型**的间隙锁（非间隙锁），使用`insert`语句的时候产生
- inserting into the same index gap need not wait for each other if they are not inserting at the same position within the gap.
  - 也是锁区间，但是互相不排斥，只要不是插入同一个**索引位置**
  - 其实可以认为**行为结果**等同于行锁，如果有间隙锁，等待间隙锁的释放

# auto-inc locks

An `AUTO-INC` lock is a special **table-level lock** taken by transactions inserting into tables with `AUTO_INCREMENT` columns. 

- 表级锁，用于保护有序自增
- simplest case, if one transaction is inserting values into the table, any other transactions must wait to do their own inserts into that table, so that rows inserted by the first transaction receive consecutive primary key values.
  - 等待第一个事务获取连续的自增值

# predicate locks for spatial indexes 

空间索引谓词锁（优化空间分析）

# [metadata locks](https://github.com/xxw1754352621/java-dev/blob/master/mysql/InnoDB/MDL.md)

元数据锁

#隐式锁和显式锁
* 显示锁(explicit lock)
    显示的加锁，在show engine innoDB status 中能够看到  ，会在内存中产生对象，占用内存  
    eg: select ... for update , select ... lock in share mode   
* 隐示锁(implicit lock)
    implicit lock 是在索引中对记录逻辑的加锁，但是实际上不产生锁对象，不占用内存空间  
    
* 哪些语句会产生implicit lock 呢？
   eg: insert into xx values(xx) 
   eg: update xx set t=t+1 where id = 1 ; 会对辅助索引加implicit lock  
* implicit lock 在什么情况下会转换成 explicit lock
  eg： 只有implicit lock 产生冲突的时候，会自动转换成explicit lock,这样做的好处就是降低锁的开销    
  eg: 比如：我插入了一条记录10，本身这个记录加上implicit lock，如果这时候有人再去更新这条10的记录，那么就会自动转换成explicit lock
* 数据库怎么知道implicit lock的存在呢？如何实现锁的转化呢？
  1. 对于聚集索引上面的记录，有db_trx_id,如果该事务id在活跃事务列表中，那么说明还没有提交，那么implicit则存在  
  2. 对于非聚集索引：由于上面没有事务id，那么可以通过上面的主键id，再通过主键id上面的事务id来判断，不过算法要非常复杂，这里不做介绍
* 锁升级，锁迁移，锁分裂（页分裂），锁合并





