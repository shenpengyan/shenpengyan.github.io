---
layout:     post
title:      "Mysql索引分析"
subtitle:   "mysql index"
date:       2019-04-19 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - Mysql
        - 索引
---


> mysql是当前使用最广泛的数据库之一，使用mysql过程中，经常会发现某个请求莫名其妙变慢了，那么，怎么进行优化呢？大多数情况下，需要通过合理利用索引来优化查询速度，今天我们就好好研究一下mysql索引。

### 索引简介

官方定义：索引（Index）是帮助Mysql高效获取数据的方式。

大家一定很好奇，索引为什么是一种数据结构，它又是怎么提高查询的速度？我们拿最常用的二叉树来分析索引的工作原理。看下面的图片：

![](https://shenpengyan.github.io/img/in-post/mysql-index/1.png)

创建索引的优势：

- 提高数据的检索速度，降低数据库IO成本，使用索引的意义就是通过缩小表中需要查询的记录的数目从而加快搜索的速度。

- 降低数据排序的成本，降低CPU消耗：索引之所以查的快，是因为先将数据排好序，若该字段正好需要排序，则正好降低了排序的成本。


创建索引的劣势：

- 占用存储空间：索引实际上也是一张表，记录了主键与索引字段，一般以索引文件的形式存储在磁盘上。

- 降低更新表的速度：表的数据发送了变化，对应的索引页需要一起变更，因此会降低更新速度。因此索引不是越多越好。
- 优质索引创建难：索引不是一成不变的，需要频繁根据用户的行为和具体的业务逻辑去创建最佳的索引。

### 索引分类

我们常说的索引一般指的是BTree（多路搜索树）结构组织的索引。其中还有聚合索引，次要索引，复合索引，前缀索引，唯一索引，统称索引。

单值索引：一个索引只包含单个列，一个表可以有多个单列索引
唯一索引：索引列的值必须唯一，但允许有空值
复合索引：一个索引包含多个列

实际开发中推荐使用复合索引，并且单表创建的索引个数建议不要超过五个

#### 基本语法

创建：

```
create [unique] index indexName on tableName (columnName...)
alter tableName add [unique] index [indexName] on (columnName...)
```

删除：

```
drop index [indexName] on tableName
```

查看：

```
show index from tableName
```

哪些情况需要建索引：
1. 主键，唯一索引
2. 经常用作查询条件的字段需要创建索引
3. 经常需要排序、分组和统计的字段需要建立索引
4. 查询中与其他表关联的字段，外键关系建立索引


哪些情况不要建索引：

1. 表的记录太少，百万级以下的数据不需要创建索引（看实际对业务的要求）
2. 经常增删改的表不需要创建索引
3. 数据重复且分布平均的字段不需要创建索引，如 true,false 之类。
4. 频繁更新的字段不适合创建索引
5. where条件里用不到的字段不需要创建索引


### 性能分析

#### MySQL自身瓶颈

MySQL自身参见的性能问题有磁盘空间不足，磁盘I/O太大，服务器硬件性能低。
1. CPU：CPU 在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时候
2. IO：磁盘I/O 瓶颈发生在装入数据远大于内存容量的时候
3. 服务器硬件的性能瓶颈：top,free,iostat 和 vmstat来查看系统的性能状态

对于大一些的公司，都有专门的dba来关注。

#### explain 分析sql语句

使用explain关键字可以模拟优化器执行sql查询语句，从而得知MySQL 是如何处理sql语句。这就是研发同学自身必须会的技能了，毕竟，没有人比你更懂业务sql。

```
+----+-------------+-------+------------+------+---------------+-----+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref  | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+-----+---------+------+------+----------+-------+
```

##### id

select 查询的序列号，包含一组可以重复的数字，表示查询中执行sql语句的顺序。一般有三种情况：

- 第一种：id全部相同，sql的执行顺序是由上至下；
- 第二种：id全部不同，sql的执行顺序是根据id大的优先执行；
- 第三种：id既存在相同，又存在不同的。先根据id大的优先执行，再根据相同id从上至下的执行。

##### select_type

- select 查询的类型，主要是用于区别普通查询，联合查询，嵌套的复杂查询
- simple：简单的select 查询，查询中不包含子查询或者union
- primary：查询中若包含任何复杂的子查询，最外层查询则被标记为primary
- subquery：在select或where 列表中包含了子查询
- derived：在from列表中包含的子查询被标记为derived（衍生）MySQL会递归执行这些子查询，把结果放在临时表里。
- union：若第二个select出现在union之后，则被标记为union，若union包含在from子句的子查询中，外层select将被标记为：derived
- union result：从union表获取结果的select

#### partitions

表所使用的分区，如果要统计十年公司订单的金额，可以把数据分为十个区，每一年代表一个区。这样可以大大的提高查询效率。

#### type(*)

这是一个非常重要的参数，连接类型，常见的有：all , index , range , ref , eq_ref , const , system , null 八个级别。

性能从最优到最差的排序：system > const > eq_ref > ref > range > index > all

对java程序员来说，若保证查询至少达到range级别或者最好能达到ref则算是一个优秀而又负责的程序员。

- all：（full table scan）全表扫描无疑是最差，若是百万千万级数据量，全表扫描会非常慢。

- index：（full index scan）全索引文件扫描比all好很多，毕竟从索引树中找数据，比从全表中找数据要快。

- range：只检索给定范围的行，使用索引来匹配行。范围缩小了，当然比全表扫描和全索引文件扫描要快。sql语句中一般会有between，in，>，< 等查询。

- ref：非唯一性索引扫描，本质上也是一种索引访问，返回所有匹配某个单独值的行。比如查询公司所有属于研发团队的同事，匹配的结果是多个并非唯一值。

- const：表示通过索引一次就可以找到，const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快，若将主键置于where列表中，MySQL就能将该查询转换为一个常量。

- system: 表只有一条记录（等于系统表），这是const类型的特列，平时不会出现，了解即可。

#### possible_keys

显示查询语句可能用到的索引(一个或多个或为null)，不一定被查询实际使用。仅供参考使用。

#### key(*)

显示查询语句实际使用的索引。若为null，则表示没有使用索引。有时候，使用的索引和预期的不太一致，那么就要仔细研究一下了。

#### key_len

显示索引中使用的字节数，可通过key_len计算查询中使用的索引长度。在不损失精确性的情况下索引长度越短越好。key_len 显示的值为索引字段的最可能长度，并非实际使用长度，即key_len是根据表定义计算而得，并不是通过表内检索出的。

#### ref
显示索引的哪一列或常量被用于查找索引列上的值。

#### rows(*)

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数，值越大越不好。

#### extra(*)

- Using filesort： 说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序操作称为“文件排序” 。出现这个就要立刻优化sql。

- Using temporary： 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序 order by 和 分组查询 group by。 出现这个更要立刻优化sql。

- Using index： 表示相应的select 操作中使用了覆盖索引（Covering index），避免访问了表的数据行，效果不错！如果同时出现Using where，表明索引被用来执行索引键值的查找。如果没有同时出现Using where，表示索引用来读取数据而非执行查找动作。

    +  覆盖索引（Covering Index） ：也叫索引覆盖，就是select 的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select 列表中的字段，而不必根据索引再次读取数据文件。

- Using index condition： 在5.6版本后加入的新特性，优化器会在索引存在的情况下，通过符合RANGE范围的条数 和 总数的比例来选择是使用索引还是进行全表遍历。

- Using where： 表明使用了where 过滤

- Using join buffer： 表明使用了连接缓存

- impossible where： where 语句的值总是false，不可用，不能用来获取任何元素

- distinct： 优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作。

#### filtered

一个百分比的值，和rows 列的值一起使用，可以估计出查询执行计划(QEP)中的前一个表的结果集，从而确定join操作的循环次数。小表驱动大表，减轻连接的次数。


通过explain的参数介绍，我们可以得知：
1. 表的读取顺序(id)
2. 数据读取操作的操作类型(type)
3. 哪些索引被实际使用(key)
4. 表之间的引用(ref)
5. 每张表有多少行被优化器查询(rows)


### 实战示例

建表语句：

```
CREATE TABLE `alchemy_coin_record_73` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_id` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '用户id',
  `coin` int(10) NOT NULL DEFAULT '0' COMMENT '金币',
  `reason` varchar(32) NOT NULL DEFAULT '' COMMENT '原因',
  `event_id` varchar(128) NOT NULL DEFAULT '' COMMENT '交易id',
  `biz_type` tinyint(4) unsigned NOT NULL DEFAULT '0' COMMENT '业务名称',
  `create_time` bigint(20) NOT NULL DEFAULT '0' COMMENT '修改时间',
  `biz_sub_type` tinyint(4) NOT NULL DEFAULT '0' COMMENT '业务子分类',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_event_id` (`event_id`),
  KEY `idx_user_id_create_time` (`user_id`,`create_time`),
  KEY `idx_user_id_reason_create_time` (`user_id`,`reason`,`create_time`)
) ENGINE=InnoDB AUTO_INCREMENT=8083612 DEFAULT CHARSET=utf8
```

查询记录列表时，

- case1: 根据user_id查create_time的列表，关注
type、key、rows、extra 命中索引idx_user_id_create_time，使用了聚合索引的最左匹配,包含2507条数据，type为ref，非唯一索引，返回所有匹配的行，Extra为Using index,直接使用索引就返回数据了。

```
 explain select create_time from alchemy_coin_record_73 where user_id = 3263973;
+----+-------------+------------------------+------+--------------------------------------------------------+-------------------------+---------+-------+------+-------------+
| id | select_type | table                  | type | possible_keys                                          | key                     | key_len | ref   | rows | Extra       |
+----+-------------+------------------------+------+--------------------------------------------------------+-------------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | alchemy_coin_record_73 | ref  | idx_user_id_create_time,idx_user_id_reason_create_time | idx_user_id_create_time | 8       | const | 2507 | Using index |
+----+-------------+------------------------+------+--------------------------------------------------------+-------------------------+---------+-------+------+-------------+
```

- case2: 根据user_id和biz_type查记录列表。命中索引idx_user_id_create_time，使用了聚合索引的最左匹配,包含2507条数据，type为ref，非唯一索引，返回所有匹配的行，Extra为Using where, 先通过索引过滤出来2507条数据，然后再使用where biz_type=1进行过滤。

```
explain select create_time from alchemy_coin_record_73 where user_id = 3263973 and biz_type = 1;
+----+-------------+------------------------+------+--------------------------------------------------------+-------------------------+---------+-------+------+-------------+
| id | select_type | table                  | type | possible_keys                                          | key                     | key_len | ref   | rows | Extra       |
+----+-------------+------------------------+------+--------------------------------------------------------+-------------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | alchemy_coin_record_73 | ref  | idx_user_id_create_time,idx_user_id_reason_create_time | idx_user_id_create_time | 8       | const | 2507 | Using where |
+----+-------------+------------------------+------+--------------------------------------------------------+-------------------------+---------+-------+------+-------------+
```

case3: 与case2相比，增加了排序功能，相关参数与case2完全一致，这里where部分和order部分都使用了索引，order 没有显示Using filesort，是因为使用了索引，那么为什么呢？因为根据where语句过滤出来的数据，都是通过索引处理过的，所以直接使用索引排序没有任何问题。顺序为：先通过索引过滤出2507条数据，然后根据根据where未命中索引的部分Using where过滤，然后，使用索引的排序处理order部分。

```
explain select create_time from alchemy_coin_record_73 where user_id = 3263973 and biz_type = 1 order by create_time desc;
+----+-------------+------------------------+------+--------------------------------------------------------+-------------------------+---------+-------+------+-------------+
| id | select_type | table                  | type | possible_keys                                          | key                     | key_len | ref   | rows | Extra       |
+----+-------------+------------------------+------+--------------------------------------------------------+-------------------------+---------+-------+------+-------------+
|  1 | SIMPLE      | alchemy_coin_record_73 | ref  | idx_user_id_create_time,idx_user_id_reason_create_time | idx_user_id_create_time | 8       | const | 2507 | Using where |
+----+-------------+------------------------+------+--------------------------------------------------------+-------------------------+---------+-------+------+-------------+
```

case4： 与case3相比，去掉where条件中的user_id, 进行了全表扫描以及filesort。type为ALL， rows为表的所有行，Extra为Using where; Using filesort。

```
explain select create_time from alchemy_coin_record_73 where  biz_type = 1 order by create_time desc;
+----+-------------+------------------------+------+---------------+------+---------+------+---------+-----------------------------+
| id | select_type | table                  | type | possible_keys | key  | key_len | ref  | rows    | Extra                       |
+----+-------------+------------------------+------+---------------+------+---------+------+---------+-----------------------------+
|  1 | SIMPLE      | alchemy_coin_record_73 | ALL  | NULL          | NULL | NULL    | NULL | 8124732 | Using where; Using filesort |
+----+-------------+------------------------+------+---------------+------+---------+------+---------+-----------------------------+
```

复杂sql的explain待补充，等遇到的吧。。。


### 性能下降的原因

从程序员的角度

1. 查询语句写的不好
2. 没建索引，索引建的不合理或索引失效
3. 关联查询有太多的join

从服务器的角度
1. 服务器磁盘空间不足
2. 服务器调优配置参数设置不合理


### 总结

1. 索引是排好序且快速查找的数据结构。其目的是为了提高查询的效率。
2. 创建索引后，查询数据变快，但更新数据变慢。
3. 性能下降的原因很可能是索引失效导致。
4. 索引创建的原则，经常查询的字段适合创建索引，频繁需要更新的数据不适合创建索引。
5. 索引字段频繁更新，或者表数据物理删除容易造成索引失效。
6. 擅用 explain 分析sql语句
7. 除了优化sql语句外，还可以优化表的设计。如尽量做成单表查询，减少表之间的关联。设计归档表等。

到这里，MySQL的索引优化分析就结束了，有什么不对的地方，大家可以提出来。如果觉得不错可以点一下赞。


写此文时参考并拷贝了部分内容自：
- http://www.cnblogs.com/itdragon/p/8146439.html
- https://mp.weixin.qq.com/s/i5ipx4dfXx4QzYAoNqyN-g






