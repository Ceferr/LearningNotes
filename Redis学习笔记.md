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

#

