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
2. 字符串常见操作
```redis
SET  key  value 			        //存入字符串键值对
MSET  key  value [key value ...] 	//批量存储字符串键值对
SETNX  key  value 		            //存入一个不存在的字符串键值对
GET  key 		                	//获取一个字符串键值
MGET  key  [key ...]	         	//批量获取字符串键值
DEL  key  [key ...] 		        //删除一个键
EXPIRE  key  seconds 		        //设置一个键的过期时间(秒)
```
3. string 应用场景
- 单值缓存

```redis
SET key value
GET key
```
- 对象缓存
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230616221130.png)
```redis
1) SET user: 1 value (json 格式数据)
2) MSET user:1: name skf user: 1 balance 1888
   MGET user:1: name user:1:balance
```

使用第二种方式更加方便, 不用进行 json 格式的解析和转换
- 分布式锁

```redis
SETNX  product: 10001  true 		//返回 1 代表获取锁成功
SETNX  product: 10001  true 		//返回 0 代表获取锁失败
。。。执行业务操作
DEL  product:10001			            //执行完业务释放锁
SET product: 10001 true  ex  10  nx	//防止程序意外终止导致死锁
```
- 计数器
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230616222220.png)

```redis
INCR article:readcount:{文章 id} 
GET article:readcount:{文章 id} 
```
- Web 集群 session 共享

Spring session + redis 实现 session 共享
- 分布式系统全局序列号

INCRBY  orderId  1000		//redis 批量生成序列号提升性能
### 2.2 hash 结构
1. Hash 常用操作

```redis
HSET  key  field  value 		        	//存储一个哈希表key的键值
HSETNX  key  field  value 		            //存储一个不存在的哈希表key的键值
HMSET  key  field  value [field value ...] 	//在一个哈希表key中存储多个键值对
HGET  key  field 				            //获取哈希表key对应的field键值
HMGET  key  field  [field ...] 		        //批量获取哈希表key中多个field键值
HDEL  key  field  [field ...] 	  	        //删除哈希表key中的field键值
HLEN  key				                    //返回哈希表key中field的数量
HGETALL  key				                //返回哈希表key中所有的键值
HINCRBY  key  field  increment 		        //为哈希表key中field键的值加上增量increment
```
2. Redis 应用场景
- 对象存储

```redis
HMSET  user  {userId}:name  zhuge  {userId}:balance  1888
HMSET  user  1:name  zhuge  1:balance  1888
HMGET  user  1:name  1:balance  
```
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230616223132.png)


![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230616223140.png)
- 电商购物车

![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230616223957.png)

1）以用户 id 为 key
2）商品 id 为 field
3）商品数量为 value
购物车操作
```redis
添加商品  hset cart: 1001 10088 1
增加数量  hincrby cart: 1001 10088 1
商品总数  hlen cart:1001
删除商品  hdel cart: 1001 10088
获取购物车所有商品  hgetall cart:1001
```
3. Hash 结构的优缺点

优点
1）同类数据归类整合储存，方便数据管理
2）相比 string 操作消耗内存与 cpu 更小
3）相比 string 储存更节省空间
缺点
过期功能不能使用在 field 上，只能用在 key 上
Redis 集群架构下不适合大规模使用
###  3. 列表 list
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230616224929.png)
1. List 常用操作

```redis
LPUSH  key  value [value ...] 		//将一个或多个值value插入到key列表的表头(最左边)
RPUSH  key  value [value ...]	 	//将一个或多个值value插入到key列表的表尾(最右边)
LPOP  key			//移除并返回key列表的头元素
RPOP  key			//移除并返回key列表的尾元素
LRANGE  key  start  stop	//返回列表key中指定区间内的元素，区间以偏移量start和stop指定
BLPOP  key  [key ...]  timeout	//从key列表表头弹出一个元素，若列表中没有元素，阻塞等待timeout秒,如果timeout=0,一直阻塞等待
BRPOP  key  [key ...]  timeout 	//从key列表表尾弹出一个元素，若列表中没有元素，阻塞等待timeout秒,如果timeout=0,一直阻塞等待
```
2. List 应用场景
- 常用数据结构

Stack (栈) = LPUSH + LPOP = FILO
Queue (队列）= LPUSH + RPOP
Blocking MQ (阻塞队列）= LPUSH + BRPOP
- 微博和微信公号消息流

![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230617115154.png)

### 4. Set 结构
1. Set 常用操作

```redis
SADD  key member  [member ...] //往集合 key 中存入元素，元素存在则忽略，若key不存在则新建
SREM  key  member [member ...]	//从集合 key 中删除元素
SMEMBERS  key					//获取集合 key 中所有元素
SCARD  key					    //获取集合 key 的元素个数
SISMEMBER  key  member			//判断 member 元素是否存在于集合 key 中
SRANDMEMBER  key  [count]		//从集合 key 中选出 count 个元素，元素不从 key 中删除
SPOP  key  [count]				//从集合 key 中选出 count 个元素，元素从 key 中删除
```
2. Set 运算操作

```redis
SINTER  key  [key ...] 			        	//交集运算
SINTERSTORE  destination  key  [key ..]		//将交集结果存入新集合 destination 中
SUNION  key  [key ..] 			        	//并集运算
SUNIONSTORE  destination  key  [key ...]	//将并集结果存入新集合 destination 中
SDIFF  key  [key ...] 				        //差集运算
SDIFFSTORE  destination  key  [key ...]		//将差集结果存入新集合 destination 中
```
3. Set 常用应用场景

- 微信抽奖小程序

![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230617120127.png)
- 微信微博点赞, 收藏, 标签

![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230617120544.png)
- 集合操作
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230617121333.png)
```redis
SINTER set1 set2 set3   { c }              // 交集
SUNION set1 set2 set3   { a,b,c,d,e }      // 并集
SDIFF set1 set2 set3    { a }              // 差集
差集是以第一个集合为基准, 减去后面集合的并集
{a,b,c} - {b,c,d,e} = {a}
```
- 集合操作实现微博微信关注模型

![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230617122916.png)
### 5. Zset 有序集合结构
```redis
help @sorted_set
```
1. Zset 常用操作

```redis
ZADD key score member [[score member]…]	//往有序集合key中加入带分值元素
ZREM key member [member …]		        //从有序集合key中删除元素
ZSCORE key member 		            	//返回有序集合key中元素member的分值
ZINCRBY key increment member	      	//为有序集合key中元素member的分值加上increment 
ZCARD key				                //返回有序集合key中元素个数
ZRANGE key start stop [WITHSCORES]	    //正序获取有序集合key从start下标到stop下标的元素
ZREVRANGE key start stop [WITHSCORES]	//倒序获取有序集合key从start下标到stop下标的元素
```
2. Zset 集合操作

```redis
Zset集合操作
ZUNIONSTORE destkey numkeys key [key ...] 	//并集计算
ZINTERSTORE destkey numkeys key [key …]	    //交集计算
```
3. Zset 应用场景

- 集合操作实现排行榜

![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230617135555.png)