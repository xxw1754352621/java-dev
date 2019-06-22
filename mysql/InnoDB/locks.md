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

# gap locks

A gap lock is a lock on a gap between index records, or a lock on the gap before the first or after the last index record

# next-key locks

A next-key lock is a combination of a record lock on the index record and a gap lock on the gap before the index record.

- 行锁防止别的事务修改或删除，GAP锁防止别的事务新增，行锁和GAP锁结合形成的的Next-Key锁共同解决了RR级别在写数据时的幻读问题。

# insert intention locks

# auto-inc locks

# predicate locks for spatial indexes 



