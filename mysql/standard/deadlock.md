# [deadlock](http://hedengcheng.com/?p=771#_Toc374698318)

## [原子操作的【一阶段封锁】 和 事务下的【两段锁】](https://tech.meituan.com/2014/08/20/innodb-lock.html)

| 事务                 | 加锁/解锁处理                                      |
| :------------------- | :------------------------------------------------- |
| begin；              |                                                    |
| insert into test ….. | 加insert对应的锁                                   |
| update test set…     | 加update对应的锁                                   |
| delete from test ….  | 加delete对应的锁                                   |
| commit;              | 事务提交时，同时释放insert、update、delete对应的锁 |

## RC和RR隔离级别

| 隔离级别                        | 脏读（Dirty Read） | 不可重复读（NonRepeatable Read） | 幻读（Phantom Read） |
| :------------------------------ | :----------------- | :------------------------------- | :------------------- |
| 未提交读（Read uncommitted）    | 可能               | 可能                             | 可能                 |
| 已提交读（Read committed）      | 不可能             | 可能                             | 可能                 |
| **可重复读（Repeatable read）** | 不可能             | 不可能                           | 可能                 |
| 可串行化（Serializable ）       | 不可能             | 不可能                           | 不可能               |

## 当前读（悲观锁）实现

RC级别和RR级别

- 加锁
  - S锁
    - 查询
      - lock in share mode;
  - X锁
    - 查询
      - for update
    - DDL，DML语句，加锁
  - 行锁
  - 间隙锁
    - **二级索引** 
    - 主键的区间数据的操作
  - **临键锁**（行锁+间隙锁）
    - RR级别下，解决幻读
  - 表锁
    - IS 和 IX，**意向锁**
    - 没有命中索引，升级为表锁
      - **mysql服务层过滤数据后，会释放部分数据的锁**
    - MDL锁

Serialable级别

- 全部操作加锁（行锁和表级锁）
  - MDL（元数据锁），也是表锁，DDL产生
    - **当前读 都会阻塞**，因为都需要对这个数据加锁的原因

## 快照读（乐观锁）的MVCC实现

- RC级别
  - 读提交
    - 
  - 没有提交
    - 依然读取当前版本的数据
- RR级别
  - **不管是否提交，读取当前版本的数据**



## 死锁模拟

- 运行环境mysql5.7，RR隔离级别，关闭自动提交，innodb_locks_unsafe_for_binlog为OFF，开启区间锁

- 网上文章因版本不同，请注意

- 等待该行 的 **X锁**释放超时，**Lock wait timeout exceeded; try restarting transaction** 

  - 模拟01
    - session01 事务1: select *from  test where id =1 for update;
    - session02 事务1: select *from  test where id =1 for update;
  - 模拟02（共享间隙锁）
    -  事务1：delete from test  where id = 5，不存在的id，【3,5】有共享间隙锁;
    -  事务2：insert test(id,name) value (3,3)，执行这句的时候，报错
  - 模拟03（主键原因 临键锁降级为行锁）
    - 事务1：delete from test  where id = 6，id存在
    - 事务2：insert test(id,name) value (3,3)，因为name不是索引，不会加间隙锁，正常执行，区间没有**共享**间隙锁

- 死锁，**Deadlock found when trying to get lock; try restarting transaction** 

  - 主要原因：**S锁和意向锁，间隙锁，这三种兼容性锁**

    - #### 意向锁 Intention Locks

      - To make locking at multiple granularity levels practical, `InnoDB` uses [intention locks](https://dev.mysql.com/doc/refman/5.5/en/glossary.html#glos_intention_lock). Intention locks are table-level locks that indicate which type of lock (shared or exclusive) a transaction requires later for a row in a table.

    - 意向gap锁 insert intention lock（不同事务兼容）

      - An insert intention lock is a type of gap lock set 
      - The transaction takes an insert intention lock while it **waits to obtain an exclusive lock.**

  - 对该行都获取了兼容性锁，在获取X的时候，互相等待对象释放 **兼容性锁**

    - 数据准备，主键 id，数据1,2,6,10，**区间【3,5】，【7,9】没有数据**
    - 模拟 01（行锁）,不提交事务

      - session01 事务1：select *from  test where id =1 LOCK IN SHARE MODE;
      - session02 事务2： select *from  test where id =1 LOCK IN SHARE MODE;
      - session01 事务1：UPDATE test SET NAME="a1" WHERE id =1;
      - session02 事务2：UPDATE test SET NAME="a1" WHERE id =1;执行这句的时候，报错，事务2回滚
    - 模拟02（对二级索引操作的共享**间隙锁**，并发）**lock_mode X locks gap** 

      - session01 事务1：BEGIN; DELETE FROM test WHERE NAME =6;【3,5】间隙锁
      - session02 事务2：BEGIN; DELETE FROM test WHERE NAME =2;【3,5】间隙锁
      - session01 事务1：INSERT INTO test(name)VALUE(5);
      - session02 事务2：INSERT INTO test(name)VALUE(3);执行这句的时候，报错，**事务2线程中断回滚**，事务1执行成功
        - 获取 意向排他锁，试图获取**排他锁**
    - 模拟03：
      - [唯一性检查](https://yq.aliyun.com/articles/198655)
      - [其他](https://cloud.tencent.com/developer/article/1056372)
    - 模拟04：
      - 并发插入
        -  事务1：INSERT INTO test(name)VALUE(15);
        -  事务2：INSERT INTO test(name)VALUE(17);
        -  事务1：INSERT INTO test(name)VALUE(17);等待 2 的排他锁
        -  事务2：INSERT INTO test(name)VALUE(15);等待1 的排他锁，**检查到死锁**
      - 并发插入
        -  事务1：INSERT INTO test(name)VALUES(15)，(15);第二次，产生nest-key锁（10,15】
        -  事务2：INSERT INTO test(name)VALUE(17)，(17);第二次，产生nest-key锁（17，+00】
        -  事务1：INSERT INTO test(name)VALUE(18);等待 2 的nest-key锁
        -  事务2：INSERT INTO test(name)VALUE(14);等待1 的nest-key锁
    - 方案：
      - 增加delete字段，不直接删除