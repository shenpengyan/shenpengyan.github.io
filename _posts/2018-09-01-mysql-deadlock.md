---
layout:     post
title:      "Mysql数据库死锁分析相关概念"
subtitle:   "mysql deadlock"
date:       2018-09-01 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - mysql
        - 死锁
---



参考博客：

mysql死锁问题分析（https://www.cnblogs.com/LBSer/p/5183300.html）

mysql insert锁机制（http://yeshaoting.cn/article/database/mysql%20insert%E9%94%81%E6%9C%BA%E5%88%B6/）

这是全网找到的比较好的两篇博客。



## 行锁：

InnoDB有三种行锁的算法：

1，Record Lock：单个行记录上的锁。

2，Gap Lock：间隙锁，锁定一个范围，但不包括记录本身。GAP锁的目的，是为了防止同一事务的两次当前读，出现幻读的情况。

3，Next-Key Lock：1+2，锁定一个范围，并且锁定记录本身。对于行的查询，都是采用该方法，主要目的是解决幻读的问题。

update、delete where为主键，只在单个行记录上锁。

不非主键索引上，会锁范围（上一个到下一个中间都会被锁）。

不在索引上，会锁整个表。

参考：https://www.cnblogs.com/zhoujinyi/p/3435982.html


## 锁的基本类型

据库上的操作可以归纳为两种：读和写。
　　多个事务同时读取一个对象的时候，是不会有冲突的。同时读和写，或者同时写才会产生冲突。因此为了提高数据库的并发性能，通常会定义两种锁：共享锁和排它锁。
　　
　　
共享锁（Shared Lock，也叫S锁）（读锁）

排他锁(Exclusive Lock，也叫X锁) （写锁）

锁行一般都是X锁

表A字段a关联了表B字段id，则表A的insert会在表B对应索引行添加S锁。
　　

## 查看一些设置

查看数据库版本：

```
select version();
```

查看数据库引擎：

```
show variables like '%engine%';
```

查看事务隔离级别：


```
select @@global.tx_isolation, @@session.tx_isolation, @@tx_isolation;
```


查看gap锁开启状态：

```
show variables like 'innodb_locks_unsafe_for_binlog';
```


查看innodb状态（包含最近的死锁日志）

```
show engine innodb status;
```


当发生锁等待时，可以通过以下命令查看各事务占用锁的情况。

```
select * from information_schema.innodb_locks;   
```

注意lock_mode字段。