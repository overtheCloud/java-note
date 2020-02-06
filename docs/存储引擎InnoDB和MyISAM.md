#### 存储引擎

插件式存储引擎是 MySQL 的重要特性，用于指定如何存储数据和索引，以及是否支持事务。常见引擎有 InnoDB 和 MyISAM。5.5 版本前默认引擎是 MyISAM，5.5 之后是 InnoDB。

```mysql
mysql> show variables like 'table_type'; # 查看默认引擎类型
mysql> show engines \G # 查看支持的数据引擎 
mysql> show variables like 'have%';  # 查看支持的数据引擎 
```

#### 常用引擎对比

| 特点         | MyISAM | InnoDB | MEMORY | MERGE | NDB  |
| ------------ | ------ | ------ | ------ | ----- | ---- |
| 存储限制     | 有     | 64TB   | 有     | -     | 有   |
| 事务安全     | -      | 支持   | -      | -     | -    |
| 锁机制       | 表锁   | 行锁   | 表锁   | 表锁  | 行锁 |
| B 树索引     | 支持   | 支持   | 支持   | 支持  | 支持 |
| 哈希索引     | -      | -      | 支持   | -     | 支持 |
| 全文索引     | 支持   | -      | -      | -     | -    |
| 集群索引     | -      | 支持   | -      | -     | -    |
| 数据缓存     | -      | 支持   | 支持   | -     | 支持 |
| 索引缓存     | 支持   | 支持   | 支持   | 支持  | 支持 |
| 数据可压缩   | 支持   | -      | -      | -     | -    |
| 空间使用     | 低     | 高     | N/A    | 低    | 低   |
| 内存使用     | 低     | 高     | 中等   | 低    | 高   |
| 批量插入速度 | 快     | 慢     | 快     | 快    | 快   |
| 支持外键     | -      | 支持   | -      | -     | -    |

#### MyISAM和InnoDB区别

MyISAM是MySQL的默认数据库引擎（5.5版之前）。虽然性能极佳，而且提供了大量的特性，包括全文索引、压缩、空间函数等，但MyISAM不支持事务和行级锁，而且最大的缺陷就是崩溃后无法安全恢复。不过，5.5版本之后，MySQL引入了InnoDB（事务性数据库引擎），MySQL 5.5版本后默认的存储引擎为InnoDB。

大多数时候我们使用的都是 InnoDB 存储引擎，但是在某些情况下使用 MyISAM 也是合适的比如读密集的情况下。（如果你不介意 MyISAM 崩溃回复问题的话）。

**两者的对比：**

1. **是否支持行级锁** : MyISAM 只有表级锁(table-level locking)，而InnoDB 支持行级锁(row-level locking)和表级锁,默认为行级锁。
2. **是否支持事务和崩溃后的安全恢复： MyISAM** 强调的是性能，每次查询具有原子性,其执行数度比InnoDB类型更快，但是不提供事务支持。但是**InnoDB** 提供事务支持事务，外部键等高级数据库功能。 具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。
3. **是否支持外键：** MyISAM不支持，而InnoDB支持。
4. **是否支持MVCC** ：仅 InnoDB 支持。应对高并发事务, MVCC比单纯的加锁更高效;MVCC只在 `READ COMMITTED` 和 `REPEATABLE READ` 两个隔离级别下工作;MVCC可以使用 乐观(optimistic)锁 和 悲观(pessimistic)锁来实现;各数据库中MVCC实现并不统一。推荐阅读：[MySQL-InnoDB-MVCC多版本并发控制](https://segmentfault.com/a/1190000012650596)
5. ......

《MySQL高性能》上面有一句话这样写到:

> 不要轻易相信“MyISAM比InnoDB快”之类的经验之谈，这个结论往往不是绝对的。在很多我们已知场景中，InnoDB的速度都可以让MyISAM望尘莫及，尤其是用到了聚簇索引，或者需要访问的数据都可以放入内存的应用。

一般情况下我们选择 InnoDB 都是没有问题的，但是某事情况下你并不在乎可扩展能力和并发能力，也不需要事务支持，也不在乎崩溃后的安全恢复问题的话，选择MyISAM也是一个不错的选择。但是一般情况下，我们都是需要考虑到这些问题的。

## InnoDB 的数据存储结构

所有数据存放在一个空间中，称为表空间。表空间又由段（segment）、区（extent）、页（page）、行（row）组成。

![](E:\gitbook\java-notes\images\InnoDB存储结构.jpg)

**页结构**如下：

![](E:\gitbook\java-notes\images\InnoDB页结构.jpg)

每个 page 大小 16k。