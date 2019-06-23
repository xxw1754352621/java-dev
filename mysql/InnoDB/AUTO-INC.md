#自增锁
 - INSERT-like 的种类
   - simple insert
   
     -  插入记录行数确认
     - 不包括： INSERT ... ON DUPLICATE KEY UPDATE.
   
   - Bulk inserts
   
     - 插入的记录行数不能马上确定的，比如： INSERT ... SELECT, REPLACE ... SELECT, and LOAD DATA
   
   - Mixed-mode inserts
   
     - 部分自增值确认
   - 这些都是simple-insert，但是部分auto increment值给定或者不给定
     
   ```mysql
     1. INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d');
     2. INSERT ... ON DUPLICATE KEY UPDATE`
   ```

- [锁模式](http://keithlan.github.io/2017/03/03/auto_increment_lock/#%E5%92%8Cauto_increment%E7%9B%B8%E5%85%B3%E7%9A%84insert%E7%A7%8D%E7%B1%BB)

  - innodb_autoinc_lock_mode = 0 (“traditional” lock mode)

  - innodb_autoinc_lock_mode = 1 (“consecutive” lock mode)

    ```properties
    原理：这是默认锁模式，当发生bulk inserts的时候，会产生一个特殊的AUTO-INC table-level lock直到语句结束，注意：（这里是语句结束就释放锁，并不是事务结束哦，因为一个事务可能包含很多语句）
    对于Simple inserts，则使用的是一种轻量级锁，只要获取了相应的auto increment就释放锁，并不会等到语句结束。
  PS：当发生AUTO-INC table-level lock的时候，这种轻量级的锁也不会加锁成功，会等待。。。。
    
  优点：非常安全，性能与innodb_autoinc_lock_mode = 0相比要好很多。
    缺点：还是会产生表级别的自增锁
    深入思考： 为什么这个模式要产生表级别的锁呢？
    因为：他要保证bulk insert自增id的连续性，防止在bulk insert的时候，被其他的insert语句抢走auto increment值。
    ```
  
  - innodb_autoinc_lock_mode = 2 (“interleaved” lock mode)
  
    ```properties
    原理：当进行bulk insert的时候，不会产生table级别的自增锁，因为它是允许其他insert插入的。来一个记录，插入分配一个auto 值，不会预分配。
    优点：性能非常好，提高并发，SBR不安全
    缺点：    
    一条bulk insert，得到的自增id可能不连续    
    SBR模式下：会导致复制出错，不一致，采用RBR模式
    ```