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