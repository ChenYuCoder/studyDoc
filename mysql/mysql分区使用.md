# Mysql分区
> 所有分区键必须在主键字段内，或者该表无主键

## 为什么要用分区
> 我们经常遇到一张表里面保存了上亿甚至过十亿的记录，这些表里面保存了大量的历史记录。 对于这些历史数据的清理是一个非常头疼事情，由于所有的数据都一个普通的表里。所以只能是启用一个或多个带where条件的delete语句去删除（一般where条件是时间）。 这对数据库的造成了很大压力。即使我们把这些删除了，但底层的数据文件并没有变小。面对这类问题，最有效的方法就是在使用分区表。最常见的分区方法就是按照时间进行分区。

>分区一个最大的优点就是可以非常高效的进行历史数据的清理。
---
## 归档策略
* HASH
> * 根据MOD(分区键，分区数)的值把数据行存储到表的不同分区中
> * 数据可以平均的分布在各个分区中
> * HASH分区的键值必须是一个INT类型的值，或是通过函数可以转为INT类型
> * HASH分区不能删除分区，所以不能使用DROP PARTITION操作进行分区删除操作；
```
PARTITION BY HASH(id) PARTITIONS 4;
```

* KEY
> * KEY分区和HASH分区相似
> * KEY分区支持除text和BLOB之外的所有数据类型的分区，而HASH分区只支持数字分区
> * KEY分区不允许使用用户自定义的表达式进行分区，KEY分区使用系统提供的HASH函数进行分区
> * 当表中存在主键或者唯一键时，如果创建key分区时没有指定字段系统默认会首选主键列作为分区字列,如果不存在主键列会选择非空唯一键列作为分区列,注意唯一列作为分区列唯一列不能为null。
```
PARTITION BY KEY(name) PARTITIONS 4;
```

* RANGE
> * 根据分区键值的范围把数据行存储到表的不同分区中
> * 多个分区的范围要连续，但是不能重叠
> * 默认情况下使用VALUES LESS THAN属性，即每个分区不包括指定的那个值
```
PARTITION BY RANGE (YEAR(time))(
PARTITION p0 VALUES LESS THAN (2017),
PARTITION p1 VALUES LESS THAN (2018),
PARTITION p2 VALUES LESS THAN (2019)
)
```

* LIST
> * 按分区键取值的列表进行分区
> * 同范围分区一样，各分区的列表值不能重复
> * 每一行数据必须能找到对应的分区列表，否则数据插入失败
```
PARTITION BY LIST (type)(
PARTITION p0 VALUES in (1, 2, 3),
PARTITION p1 VALUES in (4),
PARTITION p2 VALUES in (5),
)
```
---
## 常用命令
* 查看mysql 是否支持分区
```
show plugins;
如果存在该行则说明支持
partition	ACTIVE	STORAGE ENGINE		GPL
```
* 创建分区表
```
CREATE TABLE `test`.`test1`  (
  `id` int(10) NOT NULL,
  `time` DATETIME NOT NULL,
  `name` varchar(50) NULL
)PARTITION BY RANGE (YEAR(time))(
PARTITION p1 VALUES LESS THAN (2019),
PARTITION p2 VALUES LESS THAN (2020)
);
```

* 查看分区
```
SELECT table_name,partition_name,partition_description,table_rows FROM information_schema.`PARTITIONS` WHERE table_name = 'test1';
```

* 新建分区
```
ALTER TABLE test1 ADD PARTITION (PARTITION p3 VALUES LESS THAN(2021))
```

* 删除分区
```
ALTER TABLE test1 DROP PARTITION p3;
```
## 分区归档
> * Mysql版本>=5.7，归档分区历史数据非常方便，提供了一个交换分区的方法
> * 分区数据归档迁移条件：
> * MySQL>=5.7
> * 结构相同
> * 归档到的数据表一定要是非分区表
> * 非临时表；不能有外键约束
> * 归档引擎要是：archive
* 创建归档表
```
CREATE TABLE `test`.`test1_p1`  (
  `id` int(10) NOT NULL,
  `time` DATETIME NOT NULL,
  `name` varchar(50) NULL
  );
```
* 执行归档命令
```
ALTER TABLE test1 exchange PARTITION p1 WITH TABLE test1_p1;
```
* 修改归档表引擎（使用归档引擎的好处是：它比Innodb所占用的空间更少，但是归档引擎只能进行查询操作，不能进行写操作）
```
ALTER TABLE test1_p1 ENGINE=ARCHIVE;
```
* 删除分区
```
ALTER TABLE test1 DROP PARTITION p1;
```