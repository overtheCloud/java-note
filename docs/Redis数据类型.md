### 数据类型

| 数据类型    | 描述                                   | 数据类型         |
| ----------- | -------------------------------------- | ---------------- |
| String      | 字符串                                 | 动态字符串       |
| Hash        | 哈希，键值对，适合存储对象             | 压缩列表或字典   |
| List        | 列表，有序                             | 压缩列表或字典   |
| Set         | 无序集合，唯一                         | 整数集合或字典   |
| ZSet        | 有序集合，唯一，每个集合成员有一个分数 | 压缩列表或跳跃表 |
| HyperLogLog | 基数统计                               |                  |



### 数据结构

| 数据结构            |
| ------------------- |
| 动态字符串          |
| 链表                |
| 字典                |
| 跳跃表（zskiplist） |
| 整数集合（intset）  |
| 压缩列表（ziplist） |

### 常用命令

#### String 

- `SET key value` 
- `GET key`
- `GETRANGE key start end` 返回 key 中字符串值的子字符
- `GETSET key value` 将给定 key 的值设为 value ，并返回 key 的旧值
- `MGET key1 [key2]` 获取多个给定 key 的值
- `SETEX key seconds value` 设置 key 和 value，并指定过期时间，单位秒
- `PSETEX key milliseconds value` 设置 key 和 value，并指定过期时间，单位毫秒
- `SETNX key value` key 不存在时设置 key 和 value
- `STRLEN key` 返回 key 对应 value 的长度
- `MSET key1 value1 [key2 value 2]` 设置多个 key 和 value
- `MSETNX key1 value1 [key2 value 2]` 当所有给定 key 都不存在时设置多组 key 和 value
- `INCR key` 将 key 对应 value 值加一
- `INCRBY key increment` 给 key 对应 value 值加上指定**整数**
- `INCREBYFLOAT key increment` 给 key 对应 value 值加上指定**浮点数**
- `DESC key` 将 key 对应 value 值减一
- `DESCBY key increment` 给 key 对应 value 值减去指定**整数**
- `APPEND key value` 如果 key 存在且时一个字符串，将给定 value 追加到原来的值的末尾

#### Hash

- `HDEL key field1 [field2]` 删除一个或多个哈希表字段
- `HEXISTS key field` 查看指定哈希中给定字段是否存在
- `HGET key field` 获取哈希中给定字段的值
- `HGETALL key` 获取哈希中所有字段和值
- `HINCRBY key field increment` 为哈希中给定字段的整数值加上增量
- `HKEYS key` 获取哈希表中的所有字段
- `HLEN key ` 获取哈希中字段数量
- `HMGET key field1 [field2]` 获取哈希中给定的多个字段的值
- `HMSET key field1 value1 [field2 value2]` 将多个键值对插入哈希中
- `HSET key field value` 为哈希中指定字段赋值
- `HSETNX key field value` 哈希中字段不存在时设置字段的值
- `HVALS key` 获取哈希中所有的值
- `HSCAN key cursor [MATCH pattern] [COUNT count] ` 迭代哈希表中的键值对

#### List

- `BLPOP key1 [key2] timeout` 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
- `BRPOP key1 [key2] timeout` 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
- `BRPOPLPUSH source destination timeout` 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
- `LINDEX key index` 通过索引获取列表中的元素
- `LINSERT key BEFORE|AFTER destKey value` 在列表的元素前或者后插入元素
- `LLEN key` 获取长度
- `LPOP key` 移出并返回第一个元素
- `LPUSH key value1 [value2]` 将一个或多个值插到列表头部
- `LPUSH key value` 将一个值插入列表头部
- `LRANGE key start stop` 获取范围内的元素
-  `LREM key count value` 移出列表元素，count  > 0 从表头向表尾搜索移出 count 个元素， count = 0 移出全部相等的元素
- `LSET key index value`  通过索引设置列表元素的值
- `LTRIM key start stop` 让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除
- `RPOP key` 移出并返回列表最后一个元素
- `RPOPLPUSH source destination` 移出列表最后一个元素并添加到另一个列表后返回
- `PRUSH key value1 [value2]`  在列表尾部中添加一个或多个值
- `RPUSHX key value ` 为已存在的列表添加值

#### Set

- `SADD key member1 [member2]` 向集合添加一个或多个成员
- `SCARD key` 获取集合的成员数
- `SDIFF key1 [key2]` 返回给定所有集合的差集
- `SUNION key1 [key2]` 返回所有给定集合的并集
- `SINTER key1 [key2]` 返回给定所有集合的交集
- `SDIFFSTORE destination key1 [key2]` 返回给定所有集合的差集并存储在 destination 中
- `SINTERSTORE destination key1 [key2]` 返回给定所有集合的交集并存储在 destination 中
- `SUNIONSTORE destination key1 [key2]` 所有给定集合的并集存储在 destination 集合中
- `SISMEMBER key member` 判断 member 元素是否是集合 key 的成员
- `SMEMBERS key`  返回集合中的所有成员
- `SMOVE source destination member` 将 member 元素从 source 集合移动到 destination 集合
- `SPOP key` 移除并返回集合中的一个随机元素
- `SRANDMEMBER key [count]` 返回集合中一个或多个随机数
- `SREM key member1 [member2]` 移除集合中一个或多个成员
- `SSCAN key cursor [MATCH pattern\] [COUNT count]` 迭代集合中的元素

#### ZSet

- `ZADD key score1 member1 [score2 member2]` 向有序集合添加一个或多个成员，或者更新已存在成员的分数
- `ZCARD key` 获取有序集合的成员数
- `ZCOUNT key min max` 计算在有序集合中指定区间分数的成员数
- `ZINCRBY key increment member` 有序集合中对指定成员的分数加上增量 increment
- `ZINTERSTORE destination numkeys key [key ...]` 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
- `ZLEXCOUNT key min max` 在有序集合中计算指定区间分数内成员数量
- `ZRANGE key start stop [WITHSCORES]` 通过索引区间返回有序集合成指定区间内的成员
- `ZRANGEBYLEX key min max [LIMIT offset count]` 通过排名区间返回有序集合的成员
- `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]` 通过分数返回有序集合指定区间内的成员
- `ZRANK key member` 返回有序集合中指定成员的索引
- `ZREM key member [member ...]` 移除有序集合中的一个或多个成员
- `ZREMRANGEBYLEX key min max` 移除该集合中成员介于 `min` 和 `max` 范围内的所有元素
- `ZREMRANGEBYRANK key start stop` 移除有序集合中给定的排名区间的所有成员
- `ZREMRANGEBYSCORE key min max` 移除有序集合中给定的分数区间的所有成员
- `ZREVRANGE key start stop [WITHSCORES]` 返回有序集中指定区间内的成员，通过索引，分数从高到底
- `ZREVRANGEBYSCORE key max min [WITHSCORES]` 返回有序集中指定分数区间内的成员，分数从高到低排序
- `ZREVRANK key member` 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
- `ZSCORE key member` 返回有序集中，成员的分数值
- `ZUNIONSTORE destination numkeys key [key ...]` 计算给定的一个或多个有序集的并集，并存储在新的 key 中
- `ZSCAN key cursor [MATCH pattern] [COUNT count]` 迭代有序集合中的元素（包括元素成员和元素分值）

#### HyperLogLog

- `PFADD key value` 增加计数
- `PFCOUNT key` 获取计数
- `PFMERGE destKey key1 [key2]` 将多个计数值累加形成新的计数值