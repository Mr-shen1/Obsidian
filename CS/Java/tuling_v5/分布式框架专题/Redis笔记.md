---
tags: tuling redis
aliases: 
---
# Redis 笔记
##  1. Redis 的安装
1. 安装 gcc 
``` linux
yum install gcc
```
2. 下载 redis
```linux
http://download.redis.io/releases/redis-5.0.14.tar.gz
```
3. 解压压缩包
```linux
tar -vxf redis-5.0.14.tar
```
4. make、install
```linux
cd redis-5.0.14
make install
```
5. 修改配置
```linux
daemonize yes #后台启动 
protected‐mode no #关闭保护模式，开启的话，只有本机才可以访问redis 17 
# 需要注释掉bind 
# bind 127.0.0.1（bind绑定的是自己机器网卡的ip，如果有多块网卡可以配多个ip，代表允许客户 端通过机器的哪些网卡ip去访问，内网一般可以不配置bind，注释掉即可）
```
6. 启动服务以及验证是否启动成功
```linux
redis‐server redis.conf
ps ‐ef | grep redis
```
7. 进入 redis 客户端
```linux
redis‐cli
```
8. 退出客户端
```linux
quit
```
9. 退出 redis 服务
```linux
pkill redis‐server 
kill 进程号 
redis‐cli shutdown
```

