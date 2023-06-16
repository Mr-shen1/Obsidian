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
src/redis‐server redis.conf
ps ‐ef | grep redis
```
7. 进入 redis 客户端
```linux
src/redis‐cli
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

## 2. Redis 数据结构
### 2.1 字符串 string
1. 查看数据类型的使用**方法**
help @string

![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230616220205.png)
字符串常见操作
```redis
SET  key  value 			        //存入字符串键值对
MSET  key  value [key value ...] 	//批量存储字符串键值对
SETNX  key  value 		            //存入一个不存在的字符串键值对
GET  key 		                	//获取一个字符串键值
MGET  key  [key ...]	         	//批量获取字符串键值
DEL  key  [key ...] 		        //删除一个键
EXPIRE  key  seconds 		        //设置一个键的过期时间(秒)
```
string 应用场景
- 