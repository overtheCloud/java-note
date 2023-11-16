# 1 MySQL 架构与历史
## 1.1 MySQL 逻辑架构
![images/MySQL服务器架构.png]
## 1.2 并发控制
 - 读写锁：
	 共享锁（shared lock）| 读锁（read lock）
	 排他锁（exclusive lock）| 写锁（write lock）
 - 锁粒度：
	 表锁（table lock）
	 行锁（row lock）
## 1.3 事务
事务问题：
- 脏读(Dirty Read)
	一个事务读到另一个事务未提交的更新数据
- 不可重复读
	一个事务两次读同一行数据，可是这两次读到的数据不一样
- 幻读
	一个事务执行两次查询，但第二次查询比第一次查询多出了一些数据行
- 加锁读
概念：
#### 原子性 Atomicity 
#### 一致性 Consistency
#### 隔离性 Isolation
#### 持久性 Durability
### 1.3.1 隔离级别
- Read Uncommited
	事务中的未提交修改对其他事务可见
- Read Commited
	事务中的未提交修改对其他事务不可见，导致不可重复读问题
- Repeated Read
	同一个事务多次读取同样的记录结果一致，默认的隔离界别
	MVCC 是如何解决幻读的？
- Serializable
	强制事务串行执行，避免幻读
| 隔离级别|脏读可能性|不可重复读可行性|幻读可能性|加锁读|
|---|---|---|---|---|
|READ UNCOMMITED|Yes| Yes| Yes | No|
|READ COMMITED| No|Yes|Yes|No|
|REPEATABLE|No|No|Yes|No|
|SERIALIZABLE|No|No|No|Yes|
### 1.3.2 死锁
两个及以上事务对同一资源相互占用。例：
```
事务1：
START TRANSACTION;
UPDATE TABLE SET NAME = '1' WHERE ID = 1;
UPDATE TABLE SET NAME = '2' WHERE ID = 2;
COMMIT;
事务2：
START TRANSACTION;
UPDATE TABLE SET NAME = '1' WHERE ID = 2;
UPDATE TABLE SET NAME = '2' WHERE ID = 1;
COMMIT;
```
事务1和事务2都执行完第一行开始执行第二行时发生死锁。
解决方式：
- 锁超时后自动解锁
- InnoDB 将持有行级排他锁最少的事务回滚
### 1.3.3 事务日志
预写式日志（Write-AHead Logging）：使用事务日志，存储引擎修改表数据时，先复制数据，然后修改复制的数据并记录修改行为到事务日志，最后靠后台将复制的数据刷到磁盘。事务日志采用追加方式，写日志操作是顺序I/O，效率高。修改数据需要写两次磁盘（写日志和刷数据）；
如果数据修改已经记录到事务日志并持久化，数据还未刷回磁盘，此时系统崩溃，存储引擎重启后可以自动恢复丢失的数据。