---
tags: tuling redis 持久化 主从 
aliases: 
---

# 1 . 持久化
## 1.1 RDB
RDB 持久化是以生成 RDB 快照文件的方式，将内存中的数据持久化到磁盘中。
- 触发方式:

1. 对 Redis 进行配置，“N 秒内数据至少有 M 个改动”，如果这一条件被满足，就会触发持久化。

例如：“60 秒内至少有 1000 个键被改动”
```javascript
# save 60 1000 // 注释掉即关闭RDB持久化
```
2. 进入 Redis 客户端执行 save 或 bgsave 命令（**会将内存中所有的数据持久化**）

- bgsave 借助了操作系统的写时复制技术 (Copy-On-Write)
> Redis 借助操作系统提供的写时复制技术（Copy-On-Write, COW），在生成快照的同时，依然可以正常处理写命令。简单来说，bgsave 子进程是由主线程 fork 生成的，可以共享主线程的所有内存数据。 bgsave 子进程运行后，开始读取主线程的内存数据，并把它们写入 RDB 文件。此时，**如果主线程对这些数据也都是读操作，那么，主线程和 bgsave 子进程相互不影响。但是，如果主线程要修改一块数据，那么，这块数据就会被复制一份，生成该数据的副本。** 然后，bgsave 子进程会把这个副本数据写入 RDB 文件，而在这个过程中，主线程仍然可以直接修改原来的数据。

- save 和 bgsave 的比较

|命令|save|bgsave|
|---|---|---|
|IO类型|同步|异步|
|是否阻塞redis其他命令 |是|否(在生成子进程执行调用fork函 数时会有短暂阻塞)|
|复杂度|O(n)|O(n)|
|优点|不消耗额外内存|不阻塞客户端命令|
|缺点|阻塞客户端命令|需要fork子进程，消耗内存|

## 1.2 AOF
- 概述

AOF 是以文件追加写的方式持久化。
当执行完**修改**命令后，Reds 会将该命令记录写入记录文件 appendonly.aof 中
如：执行命令“set skf 666 ex 60”，aof 文件会记录如下数据 (resp 协议格式数据，星号后面的数字代表命令有多少个参数，$号后面的数字代表这个参数有几个字符)
```javascript
*3
$3
set
$3
skf
$3
666
$9
PEXPIREAT
$3
skf
$13
1687972682677
```
**注意**：过期时间记录的是时间戳
- 打开方式
```JavaScript
appendonly yes
```
- 配置方式
```JavaScript
appendfsync always：每次有新命令追加到 AOF 文件时就执行一次 fsync ，非常慢，也非常安全。
appendfsync everysec：每秒 fsync 一次，足够快，并且在故障时只会丢失 1 秒钟的数据。
appendfsync no：从不 fsync ，将数据交给操作系统来处理。更快，也更不安全的选择。
```
默认并且推荐的配置为 **everysec**
- AOF 重写

AOF 文件中可能又太多没用的指令，所以会定期根据**内存的最新数据**重新生成 AOF 文件
如以下指令：
```JavaScript
127.0.0.1:6379> incr readcount 
(integer) 1 
127.0.0.1:6379> incr readcount 
(integer) 2 
127.0.0.1:6379> incr readcount 
(integer) 3 
127.0.0.1:6379> incr readcount 
(integer) 4 
127.0.0.1:6379> incr readcount 
(integer) 5
```
重写后的 AOF 文件
```JavaScript
*3 
$3 
SET 
$2 
readcount 
$1 
5
```
根据以下两个配置自动重新频率
```JavaScript
# auto-aof-rewrite-min-size 64mb //aof文件至少要达到64M才会自动重写，文件太小恢复速度本来就很快，重写的意义不大 
# auto-aof-rewrite-percentage 100 //aof文件自上一次重写后文件大小增长了100%则再次触发重写
```
## 1 .3 RDB 和 AOF 两种方式的对比

|命令|RDB|AOF|
|---|---|---|
|启动优先级|低|高|
|体积|小|大|
|恢复速度|快|慢|
|数据安全性|容易丢数据|根据策略决定|

## 1.3 Redis 4.0 混合持久化
- 通过如下配置可以开启混合持久化(**必须先开启aof**)：
```JavaScript
# aof-use-rdb-preamble yes
```
如果开启了混合持久化，**AOF 在重写时**，不再是单纯将内存数据转换为 RESP 命令写入 AOF 文件，而是将重写**这一刻之前**的内存做 RDB 快照处理，并且将 RDB 快照内容和**增量的** AOF 修改内存数据的命令存在一起，都写入新的 AOF 文件。于是在 Redis 重启的时候，可以先加载 RDB 的内容，然后再重放增量 AOF 日志就可以完全替代之前的 AOF 全量文件重放，因此重启效率大幅得到提升。
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230630203227.png)

## 1.4 Redis 数据备份策略
1. 写 crontab 定时调度脚本，每小时都 copy 一份 rdb 或 aof 的备份到一个目录中去，仅仅保留最近48小时的备份
2. 每天都保留一份当日的数据备份到一个目录中去，可以保留最近1个月的备份
3. 每次copy备份的时候，都把太旧的备份给删了
4. 每天晚上将当前机器上的备份复制一份到其他机器上，以防机器损坏
# 2. 主从哨兵架构
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230701095749.png)
