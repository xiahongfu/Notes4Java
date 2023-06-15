## MVCC
在内部，InnoDB为每个记录都加上了三个隐藏字段：
* 6字节的DB_TRX_ID：最近一个对这条记录执行**插入或更新**操作的事务ID。删除在内部被视为更新，只是将记录中的某个标志位设置成已删除即可。
* 7字节的DB_ROLL_PTR：回滚指针。回滚指针指向写在rollback segment的undo log。undo log保存了记录回滚到更新前的所需要的数据。
* 6字节的DB_ROW_ID：如果表中没有主键或者唯一索引，那么数据库就会用DB_ROW_ID生成聚簇索引。否则DB_ROW_ID不会出现在索引中。

InnoDB通过read view和undo log实现MVCC。具体的没有了解，可以参考书上p13有个直观概念。

## 锁

### 共享锁与独占锁
* 共享锁（shared lock）：持有锁的事务可以读取记录
* 独占锁（exclusive lock）：持有锁的事务可以修改和删除记录

### 意向锁（Intention Locks）
意向锁是表级锁。用来表示表中某些记录被加了那种锁。有以下两种意向锁：
* 意向共享锁(IS)：事务希望在某些记录上设置共享锁。
* 意向独占锁(IX)：事务希望在某些记录上设置独占锁。

规则：
* 事务必须先获取IS或更强的锁，才能获取记录上的共享锁
* 事务必须先获取IX，才能获取记录上的独占锁
* IS锁与IX锁不冲突

意向锁不会阻塞任何锁，唯一的作用就是表明表中的某些记录被加了（或者即将被加）某种锁。

### 记录锁
锁定一条索引记录。

### 间隙锁
锁住记录间的间隙。在锁定期间不允许其它事务在这个间隙中插入数据。间隙锁的存在时为了防止幻读问题。间隙锁可以共存，也就是说，一个事务持有了某个间隙锁，**不会阻塞其它事务持有间隙锁**，只会阻塞其它事务在这个间隙中插入数据。

time1: 事务A持有(a,b)之间的间隙锁
time2: 事务B持有(a,b)之间的间隙锁
time3: 事务A希望在(a,b)之间插入一条数据，发现该间隙被锁了。则事务A在(a, b)之间持有一个插入意向锁，阻塞等待其它事务释放锁。
time4: 事务B希望在(a,b)之间插入一条数据，发现该间隙被锁了。则事务B在(a, b)之间持有一个插入意向锁，阻塞等待其它事务释放锁。

### 临键锁
就是**记录锁加**上该条记录前面的间隙的**间隙锁**。

### 插入意向锁
由insert语句在插入之前设置的一种特殊的间隙锁。插入意向锁和间隙锁互斥，和其它的插入意向锁不互斥。

## 事务模型
### 事务隔离级别
**读未提交（read uncommitted）**
普通的select语句会读取其它事务未提交的更改。会导致**脏读**。
其它语句和RC隔离级别类似。

**读已提交（read committed）**
* 每个一致性读（普通的select）都使用最新的snapshot。
* 锁定读（select for update or for share）、update、delete：**仅使用记录锁**。间隙锁仅用于外键约束检查和重复key检查。由于不使用间隙锁，因此可能会导致幻读。

RC 隔离级别的好处
* 对于update和delete语句，由于InnoDB只持有匹配行的记录锁，不匹配的行的记录锁在where语句执行完成后就释放了，因此**降低了死锁概率**。 
* 对于update语句，如果一条记录已经被锁定，InnoDB会采用半一致性读（semi-consistent read）。即返回这条记录的最新版本给MySQL，MySQL判断这条记录是否匹配where子句的条件，如果匹配，那么才需要加锁，否则不需要加锁。

**可重复读（repeatable read）**
是InnoDB默认隔离级别。
* 对于一致性读（普通select语句），同一个事务的普通select语句只有第一个会建立snapshot，后续所有的一致性读都会使用第一个select语句建立的snapshot。通过这种方式防止幻读发生（但是实际上仍然有发生的可能，参考下面一致性非锁定读部分）。
* 对于阻塞读、update语句、delete语句：使用间隙锁和记录锁来锁定区间，防止幻读。

**并行化（serializable）**
与RR隔离级别类似。但是区别在于，如果autocommit被禁止，InnoDB会将普通select语句隐式替换成`select ... for share`。如果autocommit被允许，那么一个select语句就是一个事务，因此使用一致性非锁定读方式读取也不会产生一致性问题。



### 一致性非锁定读（Consistent Nonlocking Reads）
一致性读：一致性读是InnoDB在RC和RR隔离级别下，**处理select语句的默认方式**。因为一致性读不是通过加锁实现一致性的，因此提高了并行性。**InnoDB向查询（query）提供当前时间节点的数据库快照（snapshot）来实现一致性读**。query只能看见在当前事务开始之前提交的所有更改，不会显示在当前事务开始之后提交的更改，也不会显示未提交的更改。**唯一的例外是**：事务B在事务A开始之后插入了某些行，事务A对事务B插入的这些行进行了update之后，事务A可以看见这些行，事务A也可以使用delete操作删除这些行。
```sql
SELECT COUNT(c1) FROM t1 WHERE c1 = 'xyz';  
-- 返回0：匹配到了0行
DELETE FROM t1 WHERE c1 = 'xyz';  
-- 可能会删除若干匹配行：会删除其它事务提交过的行

SELECT COUNT(c2) FROM t1 WHERE c2 = 'abc';
-- Returns 0: no rows match.
UPDATE t1 SET c2 = 'cba' WHERE c2 = 'abc';
-- Affects 10 rows: another txn just committed 10 rows with 'abc' values.
SELECT COUNT(c2) FROM t1 WHERE c2 = 'cba';
-- Returns 10: this txn can now see the rows it just updated.
```

RR隔离级别：RR隔离级别下，在第一个select语句建立snapshot。后面的select语句都使用第一个select语句建立的snapshot。
RC隔离级别：RC隔离级别下，在每个select语句都重建snapshot。

一致性非锁定读的目的是通过非锁定的方式实现同一个事务中的一致性。也就是说希望通过非锁定的方式防止**幻读**现象发生。

### 锁定读（Locking Reads）
如果在一个事务中查询了一些数据，然后在同一个事务中插入或更新相关数据，那么普通的select语句不能提供有效的保护。因为其它事务可以更新或删除刚刚查询的相关行。
InnoDB提供了两种锁定读的方式，加的锁在commit或者rollback后会被释放：

* select ... for share：对查询到的行加共享锁。
* select ... for update：对查询到的行加排它锁。



## 参考资料
[15.7 InnoDB Locking and Transaction Model](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-transaction-model.html)
[小林coding-锁篇](https://xiaolincoding.com/mysql/lock/mysql_lock.html)