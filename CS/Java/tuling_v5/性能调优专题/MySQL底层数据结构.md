---
tags: tuling MySQL 
aliases: 
---
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230706213411.png)


二叉树

为什么红黑树不适合作为 MySQL 的索引呢?
- 树的高度
- 范围查找

B 树和 B+树的区别?


为什么建议 InnoDB 表必须建主键, 并且推荐使用整形的自增主键?
1. 如果不自增的话, 会频繁的页分裂


联合索引
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230706214954.png)
![image.png](https://oss-picgo-skf.oss-cn-hangzhou.aliyuncs.com/ob/img/20230706220600.png)

最左前缀的原理
因为联合索引的创建就是根据创建索引的字段顺序排序构建的，假设第一个查询条件使用 age 时，由于没有使用到 name，那么此时用 age 来查询时，age 是无序的，会进行全表扫描
