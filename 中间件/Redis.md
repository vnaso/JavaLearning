---
title: Redis学习笔记
date: 2019/04/15 14:34
categories:
- 中间件
- Redis
tags:
- Redis
- 消息中间件
---

## Redis

### 简介

> Redis 是一个开源(BSD)许可的, **内存**中的数据结构存储系统. 它可以用作数据库, 缓存和消息中间件. 它支持多种类型的数据结构, 如字符串(String), 散列(hashes), 列表(lists), 集合(sets), 有序集合(sorted sets)与范围查询, bitmaps, LUA 脚本, LRU 驱动事件, 事务和不同级别的磁盘持久化. 并通过 Redis 哨兵和自动分区提供高可用性.

### 特点

- 性能极高
  1. 纯内存访问. Redis 将所有数据放在内存中, 内存响应时间大约为 100ns.
  2. 单线程. 因为每次的访问时间短, 使用单线程避免了多线程上下文切换的开销.
  3. 非阻塞多路 I/O 复用机制. 多个连接的管理在同一进程中. 

- 丰富的数据类型

  支持 String, hash, list, set, sorted set 等数据结构的存储.

- 原子性

  Redis 所有的操作命令都是原子性的. 多个操作也支持事务, 通过 MULTI 和 EXEC 指令.

- 丰富的特性
  - 持久化(主从分区)

    可以将内存中的数据保存在磁盘中, 重启的时候可以再次加载进行使用.

  - 发布订阅模式

  - LUA 脚本支持

  - 序列化支持

### 数据结构

> 在线熟悉 Redis 命令使用: <http://try.redis.io/>
>
> 命令大全: <http://www.redis.cn/commands.html>

#### Redis keys

Redis keys 是二进制安全的, 这意味着可以用任何二进制序列作为 key 值. 从简单的字符串到一个 JPEG 文件的内容都可以, 空字符串也是有效的 key 值.

关于 key 的几条建议:

1. 不建议使用太长的键值. 不仅消耗内存, 而且在数据中查找这类键值的计算成本很高.
2. 键值也不宜太短, 最好保持一定的可读性.
3. 最好坚持一种命名模式, 例如: `object-type:id:field` 是个不错的主意. 多个单词之间可以用 `.` 隔开, 如: `user:001:home.addr`

- keys [pattern]: 遍历所有 key.
- dbsize: 返回当前数据库里面的 keys 数量.
- exists: 检查 key 是否存在.
- del key: 删除指定的 key-value.
- expire key seconds: key 在 seconds 秒后过期.
- ttl key: 查看 key 剩余的存活时间.
- persist key: 去掉 key 的过期时间.
- type key: 返回 key 的类型.

#### String

String 的 value 可以是 String 也可以是 数字. 一般做一些**复杂 记数功能的缓存**.

**常用命令**:

- SET key value [EX seconds] [PX milliseconds]: 设置一个 key 的 value 值, 可选择设置超时时间.

  ```redis
  redis> SET mykey "Hello"
  OK
  redis> GET mykey
  "Hello"
  ```

- SETNX key value(SET if Not eXists): 设置一个 key 的 value 值, 返回 1 设置成功, 返回 0 设置失败.

  ```redis
  redis> SETNX mykey "Hello"
  (integer) 1
  redis> SETNX mykey "World"
  (integer) 0
  redis> GET mykey
  "Hello"
  ```

- SETEX key seconds value: 设置对应 key 的 value 值和超时时间.

  ```redis
  redis> SETNX mykey "Hello"
  (integer) 1
  redis> SETNX mykey "World"
  (integer) 0
  redis> GET mykey
  "Hello"
  ```

- MSET key value [key value ...]: 设置多个 key-value 键值对.

  ```redis
  redis> MSET key1 "Hello" key2 "World"
  OK
  redis> GET key1
  "Hello"
  redis> GET key2
  "World"
  ```

- GET key: 返回 key 的 value, 如果不存在, 返回特殊值 `nil`. 如果 value 不是 String, 返回错误.

  ```redis
  redis> GET nonexisting
  (nil)
  redis> SET mykey "Hello"
  OK
  redis> GET mykey
  "Hello"
  ```

- GETSET key value: 设置 key 的 value 并返回之前旧的 value 值. 如果 key 之前不存在, 返回 `nil`.

  ```redis
  redis> INCR mycounter
  (integer) 1
  redis> GETSET mycounter "0"
  "1"
  redis> GET mycounter
  "0"
  ```

- APPEND key value: 如果 key 存在, 会把 value 追加到原来 value 值的末尾, 如果不存在, 会首先创建一个空字符串的 key, 再执行追加操作.

  ```redis
  redis> INCR mycounter
  (integer) 1
  redis> GETSET mycounter "0"
  "1"
  redis> GET mycounter
  "0"
  ```

#### Hash

Hash 是一个 String 类型的 Field 和 Value 的映射表, Hash 特别适合用来**存储对象**. 

- HSET key field value: 设置 key 指定的哈希集中指定字段的值.

  ```redis
  redis> HSET myhash field1 "Hello"
  (integer) 1
  redis> HGET myhash field1
  "Hello"
  ```

- HMSET key field value [field value ...]: 设置 key 指定的哈希集中多个指定字段的值.

  ```redis
  redis> HMSET myhash field1 "Hello" field2 "World"
  OK
  redis> HGET myhash field1
  "Hello"
  redis> HGET myhash field2
  "World"
  ```

- HSETNX key field value: 只在 key 指定的哈希集中不存在指定字段时, 设置字段的值. 返回 1 表示成功赋值, 返回 0 表示存在该字段, 没有操作执行.

  ```redis
  redis> HSETNX myhash field "Hello"
  (integer) 1
  redis> HSETNX myhash field "World"
  (integer) 0
  redis> HGET myhash field
  "Hello"
  ```

- HVALS key: 返回 key 指定的哈希集中的所有字段的值.

  ```redis
  redis> HSET myhash field1 "Hello"
  (integer) 1
  redis> HSET myhash field2 "World"
  (integer) 1
  redis> HVALS myhash
  1) "Hello"
  2) "World"
  ```

- HGET key field: 返回 key 指定的哈希集中该字段所关联的值.

  ```redis
  redis> HSET myhash field1 "foo"
  (integer) 1
  redis> HGET myhash field1
  "foo"
  redis> HGET myhash field2
  (nil)
  ```

- HGETALL key: 返回 key 指定的哈希集中所有的字段和值. 每个字段名的下一个是它的值, 所以返回的长度是哈希集大小的两倍.

  ```redis
  redis> HSET myhash field1 "Hello"
  (integer) 1
  redis> HSET myhash field2 "World"
  (integer) 1
  redis> HGETALL myhash
  1) "field1"
  2) "Hello"
  3) "field2"
  4) "World"
  ```

- HLEN key: 返回 key 指定的哈希集包含的字段的数量.

  ```redis
  redis> HSET myhash field1 "Hello"
  (integer) 1
  redis> HSET myhash field2 "World"
  (integer) 1
  redis> HLEN myhash
  (integer) 2
  ```

- HEXISTS key field: 返回哈希集中对应的字段是否存在.

  ```redis
  redis> HSET myhash field1 "foo"
  (integer) 1
  redis> HEXISTS myhash field1
  (integer) 1
  redis> HEXISTS myhash field2
  (integer) 0
  ```

- HDEL key field [field ...]: 从 key 指定的哈希集中移除指定的字段, 不存在的将被忽略.

  ```redis
  redis> HSET myhash field1 "foo"
  (integer) 1
  redis> HDEL myhash field1
  (integer) 1
  redis> HDEL myhash field2
  (integer) 0
  ```

#### List

Redis List 基于 Linked List 实现, 是个双端链表. 可以非常快的在很大的列表上添加元素, 但如果需要快速访问集合元素, 建议使用(ZSet). List 可以做简单的消息队列的功能. 

- LPOP key: 移除并返回 key 对应的列表的第一个元素. 

  ```redis
  redis> RPUSH mylist "one"
  (integer) 1
  redis> RPUSH mylist "two"
  (integer) 2
  redis> RPUSH mylist "three"
  (integer) 3
  redis> LPOP mylist
  "one"
  redis> LRANGE mylist 0 -1
  1) "two"
  2) "three"
  ```

- LPUSH key value [value ...]: 将所有指定的值插入到存于 key 的列表的头部. 如果 key 不存在, 会在操作前创建一个空列表.

  ```redis
  redis> LPUSH mylist "world"
  (integer) 1
  redis> LPUSH mylist "hello"
  (integer) 2
  redis> LRANGE mylist 0 -1
  1) "hello"
  2) "world"
  ```

- LSET key index value: 设置 index 位置的元素值为 value.

  ```redis
  redis> RPUSH mylist "one"
  (integer) 1
  redis> RPUSH mylist "two"
  (integer) 2
  redis> RPUSH mylist "three"
  (integer) 3
  redis> LSET mylist 0 "four"
  OK
  redis> LSET mylist -2 "five"
  OK
  redis> LRANGE mylist 0 -1
  1) "four"
  2) "five"
  3) "three"
  ```

- LRANGE key start stop: 返回存储在 key 的列表里指定范围的元素.

  ```redis
  redis> RPUSH mylist "one"
  (integer) 1
  redis> RPUSH mylist "two"
  (integer) 2
  redis> RPUSH mylist "three"
  (integer) 3
  redis> LRANGE mylist 0 0
  1) "one"
  redis> LRANGE mylist -3 2
  1) "one"
  2) "two"
  3) "three"
  redis> LRANGE mylist -100 100
  1) "one"
  2) "two"
  3) "three"
  redis> LRANGE mylist 5 10
  (empty list or set)
  ```

- LINDEX key index: 返回存储在 key 里的索引为 index 的元素.

  ```redis
  redis> LPUSH mylist "World"
  (integer) 1
  redis> LPUSH mylist "Hello"
  (integer) 2
  redis> LINDEX mylist 0
  "Hello"
  redis> LINDEX mylist -1
  "World"
  redis> LINDEX mylist 3
  (nil)
  ```

- LLEN key: 返回存储在 key 里的 list 长度.

  ```redis
  redis> LPUSH mylist "World"
  (integer) 1
  redis> LPUSH mylist "Hello"
  (integer) 2
  redis> LLEN mylist
  (integer) 2
  ```

#### Set

Set 是 String 类型的无序集合, 集合中的元素时唯一的. Set 通过哈希表实现, 所以添加, 删除, 查找的复杂度都是 O(1).

- SADD key member [member ...]: 添加一个或多个指定的 member 元素到集合 key 中. 已经存在集合 key 中的元素将被忽略.

  ```redis
  redis> SADD myset "Hello"
  (integer) 1
  redis> SADD myset "World"
  (integer) 1
  redis> SADD myset "World"
  (integer) 0
  redis> SMEMBERS myset
  1) "World"
  2) "Hello"
  ```

- SCARD key: 返回集合存储的 key 的基数.

  ```redis
  redis> SADD myset "Hello"
  (integer) 1
  redis> SADD myset "World"
  (integer) 1
  redis> SCARD myset
  (integer) 2
  ```

- SDIFF key [key ...]: 返回一个集合与给定集合的差集元素.

  ```redis
  redis> SADD key1 "a"
  (integer) 1
  redis> SADD key1 "b"
  (integer) 1
  redis> SADD key1 "c"
  (integer) 1
  redis> SADD key2 "c"
  (integer) 1
  redis> SADD key2 "d"
  (integer) 1
  redis> SADD key2 "e"
  (integer) 1
  redis> SDIFF key1 key2
  1) "a"
  2) "b"
  ```

- SINTER key [key ...]: 返回指定所有集合的成员的交集.

  ```redis
  redis> SADD key1 "a"
  (integer) 1
  redis> SADD key1 "b"
  (integer) 1
  redis> SADD key1 "c"
  (integer) 1
  redis> SADD key2 "c"
  (integer) 1
  redis> SADD key2 "d"
  (integer) 1
  redis> SADD key2 "e"
  (integer) 1
  redis> SINTER key1 key2
  1) "c"
  ```

- SMEMBERS key: 返回 key 集合所有的元素.

  ```redis
  redis> SADD myset "Hello"
  (integer) 1
  redis> SADD myset "World"
  (integer) 1
  redis> SMEMBERS myset
  1) "World"
  2) "Hello"
  ```

- SUNION key [key ...]: 返回给定的多个集合的并集中的所有成员.

  ```redis
  redis> SADD key1 "a"
  (integer) 1
  redis> SADD key1 "b"
  (integer) 1
  redis> SADD key1 "c"
  (integer) 1
  redis> SADD key2 "c"
  (integer) 1
  redis> SADD key2 "d"
  (integer) 1
  redis> SADD key2 "e"
  (integer) 1
  redis> SUNION key1 key2
  1) "a"
  2) "b"
  3) "c"
  4) "d"
  5) "e"
  ```

- SISMEMBER key member: 返回成员 member 是否是存储的集合 key 的成员.

  ```redis
  redis> SADD myset "one"
  (integer) 1
  redis> SISMEMBER myset "one"
  (integer) 1
  redis> SISMEMBER myset "two"
  (integer) 0
  ```

#### Sorted Set(ZSet)

ZSet 和 Set 相比, 增加了一个权重参数 `score`, 使得集合中的元素能够按照 `score` 进行有序排列.

- ZADD key [NX|XX] [CH] [INCR] score member [score member ...]: 将所有指定成员添加到键为 key 的有序集合中. 添加时可以指定多个 score/member 对.

  ```redis
  redis> ZADD myzset 1 "one"
  (integer) 1
  redis> ZADD myzset 1 "uno"
  (integer) 1
  redis> ZADD myzset 2 "two" 3 "three"
  (integer) 2
  redis> ZRANGE myzset 0 -1 WITHSCORES
  1) "one"
  2) "1"
  3) "uno"
  4) "1"
  5) "two"
  6) "2"
  7) "three"
  8) "3"
  ```

- ZCARD key: 返回 key 的有序集元素个数.

  ```redis
  redis> ZADD myzset 1 "one"
  (integer) 1
  redis> ZADD myzset 2 "two"
  (integer) 1
  redis> ZCARD myzset
  (integer) 2
  ```

- ZCOUNT key min max: 返回有序集 key 中, score 值在 min 和 max 闭区间的成员.

  ```redis
  redis> ZADD myzset 1 "one"
  (integer) 1
  redis> ZADD myzset 2 "two"
  (integer) 1
  redis> ZADD myzset 3 "three"
  (integer) 1
  redis> ZCOUNT myzset -inf +inf
  (integer) 3
  redis> ZCOUNT myzset (1 3
  (integer) 2
  ```

- ZSCORE key member: 返回有序集 key 中, 成员 member 的 score 值.

  ```redis
  redis> ZADD myzset 1 "one"
  (integer) 1
  redis> ZSCORE myzset "one"
  "1"
  ```

- ZRANK key member: 返回有序集 key 中成员 member 的排名.

  ```redis
  redis> ZADD myzset 1 "one"
  (integer) 1
  redis> ZADD myzset 2 "two"
  (integer) 1
  redis> ZADD myzset 3 "three"
  (integer) 1
  redis> ZRANK myzset "three"
  (integer) 2
  redis> ZRANK myzset "four"
  (nil)
  ```

- ZRANGE key start stop [WITHSCORES]: 返回存储在有序集合 key 中指定返回的元素. 

  ```redis
  > zadd s 1 "one"
  (integer) 1
  > zadd s 2 "two"
  (integer) 1
  > zadd s 3 "three"
  (integer) 1
  > zadd s 4 "four"
  (integer) 1
  > ZRANGE s 0 -1
  1) "one"
  2) "two"
  3) "three"
  4) "four"
  > ZRANGE s 0 1 WITHSCORES
  1) "one"
  2) 1.0
  3) "two"
  4) 2.0
  ```
  
- ZREM key member [member ...]: 从 key 中存储的有序集中删除指定的元素成员.

  ```redis
  redis> ZADD myzset 1 "one"
  (integer) 1
  redis> ZADD myzset 2 "two"
  (integer) 1
  redis> ZADD myzset 3 "three"
  (integer) 1
  redis> ZREM myzset "two"
  (integer) 1
  redis> ZRANGE myzset 0 -1 WITHSCORES
  1) "one"
  2) "1"
  3) "three"
  4) "3"
  ```

### Redis 持久化

#### RDB 方式: 保存某个时间点的全量数据快照

Redis 默认的方式, 通过以指定的时间间隔执行数据集的时间点快照来将数据持久化到磁盘中. Redis 会单独创建(fork)一个子进程来进行持久化(BGSAVE), 会先将数据写入到一个临时文件中, 待持久化过程都结束了, 再用这个临时文件替换上次持久化的文件. 整个过程中, 主进程是不进行任何 IO 操作的, 这样确保了极高的性能.

SAVE: 阻塞 Redis 服务器进程, 直到 RDB 文件被创建完毕. 使用命令: `save`.

BGSAVE: Fork 出一个子进程来创建 RDB 文件, 不阻塞服务器进程. 使用命令: `bgsave`.

**Fork** 的作用是复制一个与当前进程一样的进程. 新进程的所有数据(变量, 环境变量, 程序计数器等)数值都和原进程一致, 但是是一个全新的进程, 并作为原进程的子进程.



![](https://i.loli.net/2019/04/27/5cc4601b145cc.png)

**自动触发 RDB 持久化的方式**:

1. 根据 redis.conf 配置里的 `SAVE m n` 定时触发(用的是 BGSAVE).
2. 主从复制时, 主节点自动触发.
3. 执行 Debug Reload.
4. 执行 Shutdown 且没有开启 AOF 持久化.

#### AOF方式: 保存写状态

Redis 默认是不使用该方式持久化的. AOF 方式的持久化, 是操作一次 Redis 数据库, 则将操作的记录(不包括查询操作)以 append 的形式追加保存到 AOF 持久化文件中, 以日志的形式来记录每个写操作. 

**创建**

![UTOOLS1556373551084.png](https://i.loli.net/2019/04/27/5cc4603020653.png)

**三种同步保存策略**:

1. `appendfsync always`: 同步持久化, 每次发生数据变更就会被立即记录到磁盘. 性能较差但数据完整性高.
2. `appendfsync everysec`: 异步操作, 每秒记录. 折中的方案.
3. `appendfsync no`: 从不同步.

**恢复**

- 正常恢复

  开启 AOF 持久化后, 重新启动 Redis 时会加载当前目录下的 AOF 文件.

- 异常恢复

  如果 Redis 非正常退出, 那么可能会导致写入 AOF 文件中的操作记录不完整. 这种情况下如果启动 Redis, 虽然不会报错, 但是 Redis 并未成功启动.

  **解决**: 

  1. 首先备份损坏的 AOF 文件.
  2. 使用 Redis 安装目录下的 `Redis-check-aof --fix <文件名>` 来修复文件.
  3. 重启 Redis.

![UTOOLS1556373582364.png](https://i.loli.net/2019/04/27/5cc4604f5beb9.png)

**日志重写解决 AOF 文件大小不断增大的问题**:

1. 调用 fork(), 创建一个子进程.
2. 子进程把新的 AOF 写到一个临时文件里, 不依赖原来的 AOF 文件.
3. 主进程持续将新的变动同时写到内存和原来的 AOF 中.
4. 主进程获取子进程重写 AOF 的完成信号, 往新 AOF 同步增量变动.
5. 使用新的 AOF 文件替换掉旧的 AOF 文件.

#### 两种持久化方式对比

##### RDB

|                             优势                             |                             劣势                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| 适合大规模的数据备份和恢复. 非常适用于灾难恢复(disaster recovery). | 数据丢失的风险大. 虽然 Redis 允许设置不同的保存点来控制保存 RDB 文件的频率, 但仍可能会丢失最后一次的临时文件中的数据. |
|       RDB 是个非常紧凑的文件. 且可以分成多个版本存储.        | RDB 需要经常 fork 子进程来保存数据集到硬盘. 当数据集比较大的时候, fork 的过程是非常耗时的, 可能会导致 Redis 不能毫秒级地响应客户端请求. |
| RDB 在保存 RDB文件时, 父进程唯一需要做的就是 fork 出一个子进程, 接下来的工作全部由子进程来做. 所以 RDB 持久化方式可以最大化 redis 性能. |                                                              |

##### AOF

|                             优势                             |                             劣势                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                  比 RDB 更细粒度地保存策略.                  | 相同数据集的数据, AOF 文件要远大于 RDB 文件, 恢复速度也慢于 RDB. |
|                 AOF 文件只对日志文件作追加.                  | AOF 运行效率要慢于 RDB, 每秒同步策略效率较好, 不同步效率与 RDB 相同. |
| 可以在 AOF 文件体积过大时, 自动地在后台对 AOF 进行重写(比如 `100条 incr` 优化为 `1条 incrby 100`). |                                                              |
| AOF 文件有序地保存了对数据库执行的所有写入操作, 以 Redis 协议的格式保存. 文件可读性较好. |                                                              |

**二者的选择**:

如果关心数据, 但仍然可以承受数分钟以内的数据丢失, 可以只使用 RDB 持久. AOF 将 Redis 执行的每一条指令追加到磁盘中, 处理巨大的写入会降低 Redis 的性能. 

数据库备份和灾难恢复: 定时生成的 RDB 快照(snapshot)非常便于进行数据库备份, 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快.

不使用 AOF, 仅靠 Master-Slave Replication 也可以实现高可用性. 且能减少大量 IO 的开销, 也减少了 rewrite 时带来的系统波动. 代价是: 如果 Master&Slave 同时倒掉, 会丢失最后一次的数据.

**Redis 支持同时开启 RDB 和 AOF, 系统重启后, Redis 会优先使用 AOF 来恢复数据, 这样丢失的数据会最少.**

### RDB-AOF 混合持久化方式

redis4.0 之后采用 RDB-AOF混合持久化方式. 开启这个功能之后, AOF 重写产生的文件将会同时包含 RDB 格式的内容和 AOF 格式的内容, 其中 RDB 格式的内容用于记录已有的数据, 而 AOF 格式的内容则用于记录最近发生了变化的数据. 

开启了混合持久化模式的 AOF 文件加载的流程如下:

1. AOF 文件开头是 RDB 的格式, 先加载 RDB 内容再加载剩余的 AOF.
2. AOF 文件开头不是 RDB 格式, 直接以 AOF 格式加载整个文件.

### Redis 事务

Redis 事务支持一次执行多个命令, 本质是一组命令的集合. 一个事务中的所有命令都会序列化, 按顺序地串行化执行而不会被其他命令插入.

#### 常用命令

- DISCARD: 取消事务, 放弃事务块内所有的命令.
- EXEC: 执行所有事务块内的命令, 并取消 WATCH 命令对所有 key 的监视.
- MULTI: 标记一个事务块的开始.
- WATCH key [key ...]: 监视一个或多个 key, 如果在事务执行之前 key 被其他命令所改动, 那么事务将被打断.
- UNWATCH: 取消 WATCH 命令对所有 key 的监视.

使用范式:

```redis
WATCH aKey
MULTI
// options
// set ...
EXEC
```

例: 实现加一操作

```redis
WATCH aKey
val = GET aKey
val = val + 1
MULTI
set aKey $val
EXEC
```

#### 特性

1. 如果事务块内的语法出错, 那么所有事务都不会被执行.

   语法出错, 如: `SET key`

2. 如果事务块内的指令结果出错, 那么出错的指令执行失败, 其他执行顺利执行.

   指令结果出错, 如: `SADD key 3`(key 不能以数字开头)

### Redis 发布订阅

Redis 的 SUBSCRIBE 命令可以让客户端订阅任意数量的频道, 每当有新信息发送到被订阅的频道时, 信息就会被发送给所有订阅该频道的客户端.

下图展示了频道 channel1, 以及订阅这个频道的三个客户端 - client1, client2, client5 之间的关系:

![UTOOLS1556457206152.png](https://i.loli.net/2019/04/28/5cc5a6f75cf92.png)

当新消息通过 PUBLISH 命令发送给频道 channel1 时, 这个消息就会发送给订阅它的三个客户端: 

![UTOOLS1556457359105.png](https://i.loli.net/2019/04/28/5cc5a78fe2f5b.png)

#### 常用命令

- PSUBSCRIBE pattern [pattern ...]: 订阅一个或多个符合给定模式的频道.
- PUBSUB subcommand [argument [argument ...]]: 查看订阅与发布系统状态.
- PUBLISH channel message: 将信息发送到指定的频道.
- PUNSUBSCRIBE [pattern [pattern ...]]: 退订所有给定模式的频道.
- SUBSCRIBE channel [channel ...]: 订阅给定的一个或多个频道的信息.
- UNSUBSCRIBE [channel [channel ...]]: 退订给定的频道.

示例: 

服务端

```redis
# 2 服务端发布消息
PUBLISH channel2 hello
# 4 服务端发布消息
PUBLISH goodNews something
```

客户端

```redis
# 1 客户端订阅频道
SUBSCRIBE channel1 channel2 channel3 *News
# 3 客户端收到发布的消息
1) "message" # 类型
2) "channel2" # 频道
3) "hello" # 内容
# 5 客户端收到发布的消息
1) "message" # 类型
2) "goodNews" # 频道
3) "something" # 内容
```

### Redis 主从复制

持久化保证了即使 Redis 服务器重启也不会丢失数据, 因为 Redis 服务重启后会将硬盘上持久化的数据恢复到内存中, 但是当 Redis 服务器的硬盘损坏了, 也可能导致数据丢失. 如果通过 Redis 的主从复制机制, 就可以避免这种单点故障, 如图:

![UTOOLS1555325721908.png](https://i.loli.net/2019/04/15/5cb4631a63085.png)

- 主 Redis 中的数据有两个副本, 即 slave1 和 slave2. 即使一台 Redis 服务器宕机, 其他两台 Redis 服务器仍可以继续提供服务. 

- 主 Redis 中的数据和从 Redis 上的数据保持实时同步, 当主 Redis 写入数据时, 通过主从复制机制会复制到两个从 Redis 服务上.

- 主从复制不会阻塞 master, 在同步数据时, master 可以继续处理 client 请求.

- 一个 Redis 可以同时是 master 和 slave. 如图:

  ![UTOOLS1555325983499.png](https://i.loli.net/2019/04/15/5cb4642025284.png)

#### 主从关系

建立关系: 

1. 从节点执行 `slave of <主机地址> <端口号>` .
2. 在配置文件中加入 `slave of <主机地址> <端口号>`.
3. 启动从节点时, 在 `redis-server` 后加入参数 `--slaveof <主机地址> <端口号>`.

解除关系: 从节点执行 `slaveof no one`.

查看关系: `info replication`

此外有几点需要注意: 

- 从节点不能进行写入.

- 如果没有开启哨兵模式, 那么主节点下线后, 从节点会不断尝试重新连接. 主机重新上线后, 从机会连接上主节点. 关系保持不变.
- 除非在配置文件中写明, 否则从节点每次重新启动都要使用 `slaveof` 命令与主节点建立联系.

#### 哨兵模式(sentinel)

哨兵模式下, 会在后台监控主节点是否故障, 如果主节点故障下线, 那么会根据投票数自动将从节点转换为主节点.

##### 使用

1. 创建 sentinel.conf 文件, 文件名不能为其他.

2. 添加配置:

   ```redis
   ## ip: 主机ip地址
   ## port: 哨兵端口号
   ## master-name: 可以自己命名的主节点名字(只能由字母A-z、数字0-9 、这三个字符".-_"组成).
   ## quorum: 当这些 quorum 个数 sentinel 哨兵认为 master 主节点失联, 那么这时客观上认为主节点失联了.
   sentinel monitor <master-name> <ip> <redis-port> <quorum>
   ```

3. 启动哨兵: `Redis-sentinel sentinel.conf`

**如果主节点在下线之后重新上线, 这时主节点会变成从节点.**

更加详细配置请参考: https://juejin.im/post/5b7d226a6fb9a01a1e01ff64

#### 复制原理

主从复制过程大体可分为 3 个阶段: 

1. 建立连接阶段

   该阶段的主要作用是在主从节点之间建立连接, 为数据同步做好准备.

   1. 保存主节点信息

      `slaveof` 是异步命令, 从节点完成对主节点 ip 和 port 的保存后, 向发送 `slaveof` 命令的客户端直接返回 OK, 实际的复制操作在这之后才开始.

   2. 建立 socket 连接

      从节点每秒 1 次调用定时复制函数 `replicationCron()`, 如果发现了有主节点可以连接, 便会根据主节点的 ip 和 port, 创建 socket 连接. 如果连接成功, 则:

      从节点: 为该 socket 建立一个专门处理复制工作的文件处理器, 负责后续的复制工作. 如接收 RDB 文件, 接收命令传播等.

      主节点: 接收到从节点的 socket 连接后(即 accept 后), 为该 socket 创建相应的客户端状态, 并将从节点看作是连接到主节点的一个客户端, **后面的步骤会以从节点向主节点发送命令请求的形式来进行**.

   3. 发送 ping 命令

      从节点成为主节点的客户端之后, 发送 ping 命令进行首次请求, 目的是: 检查 socket 连接是否可用, 以及主节点当前是否能够处理请求. 可能会出现三种情况: 

      (1) 返回 pong: 说明 socket 连接正常, 且主节点可以处理请求, 复制过程继续.

      (2) 超时: 一定时间后从节点仍未收到主节点的回复, 说明 socket 连接不可用, 则从节点断开 socket 连接, 并重连.

      (3) 返回 pong 以外的结果: 如果主节点返回其他结果, 如正在处理超时运行的脚本, 说明主节点当前无法处理命令, 则从节点断开 socket 连接, 并重连.

   4. 身份验证

      如果从节点中设置了 masterauth 选项, 则从节点需要向主节点进行身份验证; 没有设置该选项则不需要验证. 从节点通过向主节点发送 auth 命令来进行身份验证.

   5. 发送从节点端口信息

      身份验证之后, 从节点会向主节点发送其监听的端口号. 主节点将该信息保存到该从节点对应的客户端的 `slave_listening_port` 字段中.

2. 数据同步阶段

   主节点之间的连接建立以后, 便可以开始进行数据同步, 该阶段可以理解为从节点数据的初始化. 具体执行的方式是: 从节点向主节点发送 `psync` 命令, 开始同步.

   数据同步阶段是主从复制最核心的阶段, 根据主从节点当前状态的不同, 可以分为**全量复制**与**增量复制**.

   需要注意的是, 在数据同步之前, **从节点是主节点的客户端, 主节点不是从节点的客户端. 而到了这一阶段及以后, 主从节点互为客户端**. 原因在于: 在此之前, 从节点只需要响应从节点的请求即可, 不需要主动发请求, 而在数据同步阶段和后面的命令传播阶段, 主节点需要主动向从节点发送请求(如推送缓冲区的写命令), 才能完成复制.

3. 命令传播阶段

   数据同步阶段完成后, 主从节点进入命令传播阶段. 在这个阶段, 主节点将自己执行的写命令发送给从节点, 从节点接收命令并执行, 从而保证了主从节点数据的一致性.

   在命令传播阶段, 除了发送写命令, 主从节点还维持着心跳机制: PING 和 REPLCONF ACK.

   **延迟与不一致问题**

   命令传播是异步的过程, 即主节点发送写命令后并不会等待从节点的回复, 因此实际上主从节点之间很难保持实时一致性, 延迟在所难免. 数据不一致的程度与主从节点之间的网络状况, 主节点写命令的执行频率, 以及主节点的 `repl-disable-tcp-nodelay` 配置等有关.

   `repl-disable-tcp-nodelay no`: 该配置作用于命令传播阶段, 控制主节点是否禁止与从节点的 `TCP_NODELAY`, 默认 no, 即不禁止 `TCP_NODELAY`. 当设置为 yes 时, TCP 会对包进行合并从而减少带宽, 但是发送的频率会降低, 从节点数据延迟增加, 一致性变差. 具体发送频率与 Linux 的内核配置有关, 默认配置为 40ms. 当设置为 no 时, TCP 会立即将主节点的数据发送给从节点, 带宽增加而延迟变小.

   一般来说, 只有当应用对 Redis 数据不一致的容忍度较高, 且主从节点之间网络状况不好时, 才会设置为 yes.

#### 全量复制和增量复制

在 Redis2.8 以前, 从节点向主节点发送 sync 命令请求同步数据, 此时的同步方式是全量复制. 在 Redis2.8 以后, 从节点可以发送 psync 命令请求同步数据, 此时根据主从节点当前状态的不同, 同步方式可能是全量复制或增量复制.

##### 全量复制

用于初次复制或其他无法进行部分复制的情况, 将主节点中的所有数据都发送给从节点, 是一个重操作.

##### 增量复制

用于网络中断等情况后的复制, 只将中断期间主节点执行的写命令发送给从节点, 与全量复制相比更加高效. 需要注意的是: 如果网络中断时间过长, 导致主节点没有能够完整地保存中断期间执行的写命令, 则无法进行增量复制, 仍使用全量复制.

## 参考

- 面试中关于Redis的问题看这篇就够了 <https://juejin.im/post/5ad6e4066fb9a028d82c4b66>
- 搞懂Redis到底快在哪里 <https://zhuanlan.zhihu.com/p/61816273>
- Redis从入门到实践 https://juejin.im/post/5a912b3f5188257a5c608729
- redis的事务和watch <https://www.jianshu.com/p/361cb9cd13d5>

 