---
tags: tuling redis redlock  
aliases: 
---
# RedLock 存在的问题
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230702174601.png)

1. 需要新增 Redis 节点，只有超过半数以上的节点加锁成功才算成功，这样降低了 Redis 实现分布式锁的性能。
2. 节点宕机。由于持久化策略可能会丢失数据，无论是 RDB 还是 AOF，假设线程一进来，redis 1 节点加锁成功， redis 2 节点也加锁成功，这时线程一加锁成功，但是 redis 2 节点在写入后瞬间宕机了，此时将 redis 2 节点重启后也会丢失最近 1 s 的数据，相当于 redis 2 节点没有加锁了，这时如果有线程二进来，就可以加锁成功 redis 2 和 redis 3 节点，也能超过一半加锁成功。