---
title: 'Redis 集群(Redis Cluster)'
description: '简述如何使用 docker-compose 搭建 3 主 3 从的 Redis 集群'
keywords: 'redis'

date: 2023-01-05T21:46:25+08:00

categories:
  - Ops
tags:
  - Redis
---

简述如何使用 docker-compose 搭建 3 主 3 从的 Redis 集群。

<!--more-->

## 创建配置文件

### 创建挂载目录

```shell
# 递归创建 docker、redis 目录
mkdir -p /opt/docker/redis

# 进入 redis 目录下
cd /opt/docker/redis
```

### 创建集群模板配置文件

```shell
vim redis-cluster
```

文件内容:

```conf
bind 0.0.0.0
# redis端口
port ${PORT}
tcp-backlog 511
timeout 0
tcp-keepalive 300
supervised no
pidfile /var/run/redis_${PORT}.pid
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
dir /data/
repl-disable-tcp-nodelay no
slave-priority 100
maxclients 60000
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
slave-lazy-flush no
#redis 访问密码
requirepass lucas
#redis 访问Master节点密码
masterauth lucas
# 关闭保护模式
protected-mode no
# 开启集群
cluster-enabled yes
# 集群节点配置
cluster-config-file nodes_${PORT}.conf
# 超时
cluster-node-timeout 5000
# 集群节点IP host模式为宿主机IP
cluster-announce-ip 192.168.100.101
# 集群节点端口 6391 - 6396
cluster-announce-port ${PORT}
cluster-announce-bus-port 1${PORT}
# 开启 appendonly 备份模式
appendonly yes
appendfilename "appendonly.aof"
# 每秒钟备份
appendfsync everysec
# 对aof文件进行压缩时，是否执行同步操作
no-appendfsync-on-rewrite no
# 当目前aof文件大小超过上一次重写时的aof文件大小的100%时会再次进行重写
auto-aof-rewrite-percentage 100
# 重写前AOF文件的大小最小值 默认 64mb
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble no
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
# 日志配置
# debug:会打印生成大量信息，适用于开发/测试阶段
# verbose:包含很多不太有用的信息，但是不像debug级别那么混乱
# notice:适度冗长，适用于生产环境
# warning:仅记录非常重要、关键的警告消息
loglevel notice
# 日志文件路径
logfile "/data/redis.log"
```

需要修改的位置：

```conf
#redis 访问密码
requirepass lucas
#redis 访问Master节点密码
masterauth lucas
cluster-announce-ip 192.168.100.101
```

### 循环生成节点目录和配置文件

```shell
for port in `seq 6379 6381`;
do \
mkdir -p redis_${port}/conf \
&& PORT=${port} envsubst <redis-cluster> redis_${port}/conf/redis.conf \
&& mkdir -p redis_${port}/data;\
done
```

### 创建 docker-compose.yml 文件

```shell
# 进入 /docker/redis 目录下
cd opt/docker/redis

# 创建 docker-compose.yml 文件
vim docker-compose.yml
```

文件内容:

```yml
version: '3.1'
services:
  redis-node1:
    image: redis:5.0.14 # 基础镜像
    container_name: redis_6379 # 容器名称
    ports: # 映射端口，对外提供服务
      - 6379:6379 # redis的服务端口
      - 16379:16379 # redis集群监控端口
    stdin_open: true # 标准输入打开
    tty: true # 后台运行不退出
    network_mode: host # 使用host模式 必须 共享宿主机IP
    privileged: true # 拥有容器内命令执行的权限
    restart: always
    volumes:
      - /opt/docker/redis/redis_6379/conf/redis.conf:/config/redis.conf # 配置文件目录映射到宿主机
      - /opt/docker/redis/redis_6379/data:/data # 数据文件目录映射到宿主机
    command: ['redis-server', '/config/redis.conf'] # 设置服务默认的启动命令
  redis-node2:
    image: redis:5.0.14
    container_name: redis_6380
    ports:
      - 6380:6380
      - 16380:16380
    stdin_open: true
    tty: true
    network_mode: host
    privileged: true
    restart: always
    volumes:
      - /opt/docker/redis/redis_6380/conf/redis.conf:/config/redis.conf
      - /opt/docker/redis/redis_6380/data:/data
    command: ['redis-server', '/config/redis.conf']
  redis-node3:
    image: redis:5.0.14
    container_name: redis_6381
    ports:
      - 6381:6381
      - 16381:16381
    stdin_open: true
    tty: true
    network_mode: host
    privileged: true
    restart: always
    volumes:
      - /opt/docker/redis/redis_6381/conf/redis.conf:/config/redis.conf
      - /opt/docker/redis/redis_6381/data:/data
    command: ['redis-server', '/config/redis.conf']
```

## 搭建集群

### 构建 redis 容器

```shell
# 在放置 docker-compose.yml 文件的目录下执行
docker-compose -f docker-compose.yml up -d
```

### 搭建

进入任一容器：

```shell
# 进入容器（redis_6379）内部
docker exec -it redis_6379 /bin/bash

# 切换到 /usr/local/bin 路径下
cd /usr/local/bin
```

执行命令创建集群：

```shell
redis-cli -a lucas --cluster create \
192.168.100.101:6379 \
192.168.100.101:6380 \
192.168.100.101:6381 \
192.168.100.102:6379 \
192.168.100.102:6380 \
192.168.100.102:6381 \
--cluster-replicas 1
```
