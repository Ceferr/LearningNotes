# 一.Redis的安装

## 1.在docker上安装Redis

附：docker基本命令：

查看所有镜像 docker images

删除镜像(会提示先停止使用中的容器) docker rmi  镜像name/镜像id

查看所有容器 docker ps -a

查看容器运行日志 docker logs 容器名称/容器id

停止容器运行 docker stop 容器name/容器id

终止容器后运行 docker start 容器name/容器id

容器重启 docker restart 容器name/容器id

删除容器 docker rm 容器name/容器id


### 1.docker pull redis

### 2.准备data和redis.conf

官网上下载redis.conf配置文件放在/usr/local/redis/   （放哪都可以，只是为了映射docker容器的redis.conf）

一个目录/usr/local/redis/data

一个配置文件/usr/local/redis/redis.conf

- 配置文件中**bind 127.0.0.1**注释掉，否则只能本机客户端连接redis服务器
- **requirepass** 设置密码
- 是否开启保护模式，默认开启。要是配置里没有指定bind和密码。开启该参数后，redis只会本地进行访问，拒绝外部访问。**protected-mode yes**

### 3.docker启动redis

docker run -p 6379:6379 --name myredis -v /usr/local/redis/redis.conf:/etc/redis/redis.conf -v /usr/local/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes

- -p [hostport]:[containerport] 端口映射
- -v [hostfile]:[containerfile] 文件映射
- -d 后台启动
- redis redis-server /etc/redis/redis.conf 以配置文件方式启动
- --appendonly yes 开启redis持久化

#二.Redis数据类型

## 1.Redis的key

- keys *  查看key
- exists key 判断某个key是否存在
- move key db 将某个key移到某个库
- expire key 为key设置过期时间
- ttl key 查看key的剩余时间 ，-1永不过期，-2已过期
- type key 查看key的类型

## 2.String字符串

- set/get/del/append/strlen
- incr/decr/incrby/decrby(自增/自减/加/减)
- getrange(获得某个区间的值)/setrange(从第几位开始设值)
- setex(set with expire)/setnx(set if not exist)
- mset/mget/msetnx(多个)
- getset(先get再set)

## 3.List列表

- lpush/rpush/lrange
- lpop/rpop
- lindex
- llen
- lrem key n n 删除n个 n
- ltrim key 将key中指定范围的值取出来再赋给key
- rpoplpush
- lset key index value
- linsert key before/after 

## 4.Set集合

- sadd/smembers/sismember
- scard
- srem key value
- srandmember key
- spop key
- smove key1 key2
- 数学集合
  - 差集sdiff
  - 交集sinter
  - 并集sunion

## 5.Hash哈希

- hset/hget/hmset/hmget/hgetall/hdel
- hlen
- hexists key 
- hkeys/havls
- hincrby/hincrbyfloat
- hsetnx

## 6.Zset有序集合

- add/zrange
- zrangebyscore key min max('('表示不包括)
- zrem key
- zcard/zcount key score区间/zrank key values值，作用时获得下标值/zscore key 对应值，获得分数
- ZREVRANK key member 逆序获得下标值
- ZREVRANGE key start stop [WITHSCORES]
- ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT]

# 三 .Redis持久化

## 1.RDB(Redis DataBase)

​	在指定时间间隔内将内存中的数据集快照写入磁盘，也就	是Snapshot快照，它恢复时是将快照文件写入内存。

​	RDB保存的是dump.rdb文件。

## 2.AOF(Append Only File)

​	以日志的形式记录所有的写操作

​	AOF和RDB是两种恢复策略，是协同的，redis会先去加载	AOF的形式。

​	**AOF的文件重写**：AOF采用追加的方式，文件会越来越	大，当AOF文件大小超过阈值，redis会进行压缩，只保留可以恢复数据的最小指令集，可以用命令bgrewriteaof。

​	**重写原理**：fork出一条新进程来重写，便利内存中的数据，每条记录有一条set语句，重写aof时，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写一个新的aof文件。

​	**重写触发机制**：redis会记录上次重写的aof大小，默认配置是当aof文件大小是上次rewrite后大小的一倍且文件大于64m时处罚

​	AOF的几个策略

- appendonly(Y/N)
- appendfilename 指定aof文件名
- appendfsync
  - Always 同步持久化，相当于同步复制，性能较差但是完整性比较好
  - Everysec 出厂模式推荐，异步操作，每秒记录
  - No
- No-appendfsync-on-rewrite 重写时是否可以运用Appendfsync，默认no
- Auto-aof-rewrite-min-size 100  (上一次aof文件的一倍)
- Auto-aof-rewrite-percentage 64mb

# 四.Redis事务

## 1.事务常用命令

- DISCARD 取消事务
- EXEC 执行所有事务块内的命令
- MULTI 标记一个事务块的开始
- UNWATCH 取消WATCH命令对所有key的监视
- WATCH Key [key...] 监视一个(或多个)，如果在事务之前这个key被其它命令改动，那么事务被打断。

## 2.事务的5种场景

1. **正常执行** MULTI --> EXEC
2. **放弃事务** MULTI --> DISCARD
3. **全体连坐** 语法有误，出现error，那全体不执行
4. **冤头债主** 语法无误，逻辑错误的一条不执行
5. **watch监控** 监视一个(或多个)，如果在事务之前这个key被其它命令改动，那么事务被打断。

## 3.事务的3阶段

1. 开启：以MULTI开启一个事务；
2. 入队：将多个命令入队到事务中，命令不会立即执行；
3. 执行：由EXEC命令触发事务

## 4.事务的3个特性

1. 单独的隔离操作：事务中的所有命令都会序列化，按顺序地执行，事务执行的过程中不会被客户端发来的其它请求打断；
2. 没有隔离级别的概念：队列中的命令不提交就不执行，也就不存在“事务内的查询要看到事务里的更新，在事务外查询不能看到”的问题；
3. 不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然可以执行，不会发生回滚。

# 五.Redis的主从复制

## 1.是什么

​	主机数据更新后根据配置和策略，自动同步到备机的master/slave机制，master以写为主，slave以读为主

## 2.主要作用

- 读写分离
- 容灾恢复

## 3.常用的3种操作

​	slaveof ip port

1. ​	一主二从

   一台master两台slave

2. 薪火相传

   一台master挂一台slave，slave再挂slave，可以减轻master压力

3. 反客为主

   使一台slave成为master，SLAVEOF no one

## 4.复制原理

- Slave启动成功连接到master后会发一个sync命令

  Master在接到命令后启动后台的存盘进程，同收集所有接收到用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，已完成一次完全同步

- 全量复制：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。

- 增量复制：Master继续将新的收集到的修改命令依次传给slave，完成同步。

- 但只要是重新连接master，一次完全同步（全量复制）将被自动执行

## 5.哨兵模式(sentinel)

- 反客为主的自动版，能够后台监控主机是否故障，如果故障了，根据投票自动将从库转为主库
- 创建sentinel.conf:
- sentinel monitor 主机名(被监控的主机) +ip +port  n(谁的票数多n谁就是master)
- 启动哨兵: Redis-sentinel ${path}/sentinel.conf

