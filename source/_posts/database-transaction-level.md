---
title: 数据库事务隔离级别
date: 2016-09-16 22:26:07
tags: 数据库
categories: 数据库
---
  S QL:1992 事务隔离级别，InnoDB默认是可重复读的（REPEATABLE READ）。MySQL/InnoDB 提供SQL标准所描述的所有四个事务隔离级别。你可以在命令行用--transaction-isolation选项，或在选项文件里，为所有连接设置默认隔离级别。
例如，你可以在my.inf文件的[mysqld]节里类似如下设置该选项：
```sql
transaction-isolation = [ READ-UNCOMMITTED | READ-COMMITTED | REPEATABLE-READ | SERIALIZEABLE]
```
用户可以用SET TRANSACTION语句改变单个会话或者所有新进连接的隔离级别。它的语法如下：
```sql
SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL [ READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE]
```
注意：默认的行为（不带session和global）是为下一个（未开始）事务设置隔离级别。如果你使用GLOBAL关键字，语句在全局对从那点开始创建的所有新连接（除了不存在的连接）设置默认事务级别。你需要SUPER权限来做这个。使用SESSION 关键字为将来在当前连接上执行的事务设置默认事务级别。 任何客户端都能自由改变会话隔离级别（甚至在事务的中间），或者为下一个事务设置隔离级别。

查看全局和绘画事务隔离界别:
```sql
select @@global.tx_isolation;
select @@session.tx_isolation;
select @@tx_isolation;
```

### 数据库读数据问题概念
1. **脏读（Dirty Reads）**
    所谓脏读就是对脏数据（Drity Data）的读取，而脏数据所指的就是未提交的数据。也就是说，一个事务正在对一条记录做修改，在这个事务完成并提交之前，这条数据是处于待定状态的（可能提交也可能回滚），这
时，第二个事务来读取这条没有提交的数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被称为脏读
2. **不可重复读（Non-Repeatable Reads）**
    一个事务先后读取同一条记录，但两次读取的数据不同，我们称之为不可重复读。也就是说，这个事务在两次读取之间该数据被其它事务所修改
3. **幻读（Phantom Reads）**
    一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为幻读

### 数据库事务隔离级别

1. **未提交读（Read Uncommitted）**
    SELECT语句以非锁定方式被执行，所以有可能读到脏数据，隔离级别最低
2. **提交读（Read Committed）**
    只能读取到已经提交的数据。即解决了脏读，但未解决不可重复读
3. **可重复读（Repeated Read）**
    在同一个事务内的查询都是事务开始时刻一致的，InnoDB的默认级别。在SQL标准中，该隔离级别消除了不可重复读，但是还存在幻读
4. **串行读（Serializable）**
    完全的串行化读，所有SELECT语句都被隐式的转换成SELECT ... LOCK IN SHARE MODE，即读取使用表级共享锁，读写相互都会阻塞。隔离级别最高


隔离级别对照表

| 事务级别 | 脏读 | 可重复读 | 幻读 |
| -------- | ---- | ----- | ------ |
|未提交读 | <font color='red'>未解决</font> |  <font color='red'>未解决</font> |  <font color='red'>未解决</font>|
|提交读 |  <font color='green'>解决</font> |  <font color='red'>未解决</font> |  <font color='red'>未解决</font> |
|可重复读|  <font color='green'>解决</font> |  <font color='green'>解决</font> |  <font color='red'>未解决</font> |
|串行读|  <font color='green'>解决</font>|  <font color='green'>解决</font>|  <font color='green'>解决</font>|


### 举例说明各级别情况

#### 1. 脏读。脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据
```sql
session 1:
mysql> select @@global.tx_isolation;
+-----------------------+
| @@global.tx_isolation |
+-----------------------+
| REPEATABLE-READ       |
+-----------------------+
row in set (0.00 sec)

mysql> select @@session.tx_isolation;
+-----------------------+
| @@session.tx_isolation |
+-----------------------+
| REPEATABLE-READ       |
+-----------------------+
row in set (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into ttd values(1);
Query OK, 1 row affected (0.05 sec)

mysql> select * from ttd;
+------+
| id   |
+------+
|    1 |
+------+
row in set (0.00 sec)

session 2:
mysql> select @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| REPEATABLE-READ        |
+------------------------+
row in set (0.00 sec)

mysql> select @@global.tx_isolation;
+-----------------------+
| @@global.tx_isolation |
+-----------------------+
| REPEATABLE-READ   |        --------该隔离级别下(除了 read uncommitted)
+-----------------------+
row in set (0.00 sec)

mysql> select * from ttd;
Empty set (0.00 sec)              --------不会出现脏读

mysql> set session transaction isolation level read uncommitted;
Query OK, 0 rows affected (0.00 sec)

mysql> select @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| READ-UNCOMMITTED       |   --------该隔离级别下
+------------------------+
row in set (0.00 sec)

mysql> select * from ttd;
+------+
| id   |
+------+
|    1 |                                       --------READ UNCOMMITTED级别出现脏读

+------+
row in set (0.00 sec)

```
`结论`:session 2 在READ-UNCOMMITTED 下读取到session 1 中未提交事务修改的数据.

#### 2. 不可重复读：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。防止了脏读，即读取不到未提交的更改，但是可以读取到提交之后的更改。因此不能重复读，因为两次读取的结果不一致

```sql
session 1:
mysql> select @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| READ-COMMITTED         |
+------------------------+
row in set (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from ttd;
+------+
| id   |
+------+
|    1 |
+------+
row in set (0.00 sec)

session 2 :

mysql> select @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| REPEATABLE-READ        |
+------------------------+
row in set (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from ttd;
+------+
| id   |
+------+
|    1 |
+------+
row in set (0.00 sec)

mysql> insert into ttd values(2);  /也可以更新数据
Query OK, 1 row affected (0.00 sec)

mysql> select * from ttd;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
rows in set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.02 sec)

session 2 提交后,查看session 1 的结果;

session 1:

mysql> select * from ttd;
+------+
| id   |
+------+
|    1 |                             --------和第一次的结果不一样,READ-COMMITTED 级别出现了不重复读
|    2 |
+------+
rows in set (0.00 sec)
```

#### 3. 可重复读：repeatable-read级别，防止了不可重复读，但是相应的也读取不到了开始事务之后其他事务的提交。

```sql
session 1:
mysql> select @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| REPEATABLE-READ        |
+------------------------+
row in set (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from ttd;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
rows in set (0.00 sec)

session 2 :

mysql> select @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| REPEATABLE-READ        |
+------------------------+
row in set (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into ttd values(3);
Query OK, 1 row affected (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.03 sec)

session 2 提交后,查看session 1 的结果;

session 1:

mysql> select * from ttd;
+------+
| id   |
+------+
|    1 |                                      --------和第一次的结果一样,REPEATABLE-READ级别出现了重复读
|    2 |
+------+
rows in set (0.00 sec)

(commit session 1 之后 再select * from ttd 可以看到session 2 插入的数据3)
```

#### 4.幻读: 因为读取不到其他事务提交的结果，所以出现了幻读，
第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样

```sql
mysql>CREATE TABLE `t_bitfly` (
`id` bigint(20) NOT NULL default '0',
`value` varchar(32) default NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB

mysql> select @@global.tx_isolation, @@tx_isolation;
+-----------------------+-----------------+
| @@global.tx_isolation | @@tx_isolation  |
+-----------------------+-----------------+
| REPEATABLE-READ       | REPEATABLE-READ |
+-----------------------+-----------------+

实验一：

t Session A                   Session B
|
| START TRANSACTION;          START TRANSACTION;
|
| SELECT * FROM t_bitfly;
| empty set
|                             INSERT INTO t_bitfly
|                             VALUES (1, 'a');
|
| SELECT * FROM t_bitfly;
| empty set
|                             COMMIT;
|
| SELECT * FROM t_bitfly;
| empty set
|
| INSERT INTO t_bitfly VALUES (1, 'a');
| ERROR 1062 (23000):
| Duplicate entry '1' for key 1
v (shit, 刚刚明明告诉我没有这条记录的)

如此就出现了幻读，以为表里没有数据，其实数据已经存在了，傻乎乎的提交后，才发现数据冲突了。

实验二：

t Session A                  Session B
|
| START TRANSACTION;         START TRANSACTION;
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                            INSERT INTO t_bitfly
|                            VALUES (2, 'b');
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                            COMMIT;
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|
| UPDATE t_bitfly SET value='z';
| Rows matched: 2  Changed: 2  Warnings: 0
| (怎么多出来一行)
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | z     |
| |    2 | z     |
| +------+-------+
```
本事务中第一次读取出一行，做了一次更新后，另一个事务里提交的数据就出现了。也可以看做是一种幻读。
当隔离级别是可重复读，且禁用innodb_locks_unsafe_for_binlog的情况下，在搜索和扫描index的时候使用的next-key locks可以避免幻读

再看一个实验，要注意，表t_bitfly里的id为主键字段
```sql
实验三：
t Session A                 Session B
|
| START TRANSACTION;        START TRANSACTION;
|
| SELECT * FROM t_bitfly
| WHERE id<=1
| FOR UPDATE;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                           INSERT INTO t_bitfly
|                           VALUES (2, 'b');
|                           Query OK, 1 row affected
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                           INSERT INTO t_bitfly
|                           VALUES (0, '0');
|                           (waiting for lock ...then timeout)
|                           ERROR 1205 (HY000):
|                           Lock wait timeout exceeded;
|                           try restarting transaction
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                           COMMIT;
|
| SELECT * FROM t_bitfly;

| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
```
可以看到，用id<=1加的锁，只锁住了id<=1的范围，可以成功添加id为2的记录，添加id为0的记录时就会等待锁的释放
```sql
实验四：一致性读和提交读
t Session A                      Session B
|
| START TRANSACTION;             START TRANSACTION;
|
| SELECT * FROM t_bitfly;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| +----+-------+
|                                INSERT INTO t_bitfly
|                                VALUES (2, 'b');
|                                COMMIT;
|
| SELECT * FROM t_bitfly;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| +----+-------+
|
| SELECT * FROM t_bitfly LOCK IN SHARE MODE;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| |  2 | b     |
| +----+-------+
|
| SELECT * FROM t_bitfly FOR UPDATE;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| |  2 | b     |
| +----+-------+
|
| SELECT * FROM t_bitfly;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| +----+-------+
```
如果使用普通的读，会得到一致性的结果，如果使用了加锁的读，就会读到“最新的”“提交”读的结果。

本身，可重复读和提交读是矛盾的。在同一个事务里，如果保证了可重复读，就会看不到其他事务的提交，违背了提交读；如果保证了提交读，就会导致前后两次读到的结果不一致，违背了可重复读。

可以这么讲，InnoDB提供了这样的机制，在默认的可重复读的隔离级别里，可以使用加锁读去查询最新的数据（提交读）。
MySQL InnoDB的可重复读并不保证避免幻读，需要应用使用加锁读来保证。而这个加锁度使用到的机制就是next-key locks。

总结:

四个级别逐渐增强，每个级别解决一个问题。事务级别越高,性能越差,大多数环境read committed 可以用.记住4个隔离级别的特点(上面的例子);

