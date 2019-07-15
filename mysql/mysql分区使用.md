# Mysql分区(整理非原创)
> 本文档部分内容为转载，非原创

> 所有分区键必须在主键字段内，或者该表无主键
---
## 分区表的原理
> 分区表是由多个相关的底层表实现，这些底层表也是由句柄对象表示，所以我们也可以直接访问各个分区，存储引擎管理分区的各个底层表和管理普通表一样（所有的底层表都必须使用相同的存储引擎），分区表的索引只是在各个底层表上各自加上一个相同的索引，从存储引擎的角度来看，底层表和一个普通表没有任何不同，存储引擎也无须知道这是一个普通表还是一个分区表的一部分。

> 虽然每个操作都会打开并锁住所有的底层表，但这并不是说分区表在处理过程中是锁住全表的，如果存储引擎能够自己实现行级锁，如：innodb，则会在分区层释放对应的表锁，这个加锁和解锁过程与普通Innodb上的查询类似。

>分区一个最大的优点就是可以非常高效的进行历史数据的清理。
在分区表上的操作按照下面的操作逻辑进行：
* select查询：
> 当查询一个分区表的时候，分区层先打开并锁住所有的底层表，优化器判断是否可以过滤部分分区，然后再调用对应的存储引擎接口访问各个分区的数据

* insert操作：
> 当写入一条记录时，分区层打开并锁住所有的底层表，然后确定哪个分区接受这条记录，再将记录写入对应的底层表

* delete操作：
> 当删除一条记录时，分区层先打开并锁住所有的底层表，然后确定数据对应的分区，最后对相应底层表进行删除操作

* update操作：
> 当更新一条数据时，分区层先打开并锁住所有的底层表，mysql先确定需要更新的记录在哪个分区，然后取出数据并更新，再判断更新后的数据应该放在哪个分区，然后对底层表进行写入操作，并对原数据所在的底层表进行删除操作
---
## 在下面的场景中，分区可以起到非常大的作用：

* A：表非常大以至于无法全部都放在内存中，或者只在表的最后部分有热点数据，其他都是历史数据
* B：分区表的数据更容易维护，如：想批量删除大量数据可以使用清除整个分区的方式。另外，还可以对一个独立分区进行优化、检查、修复等操作
* C：分区表的数据可以分布在不同的物理设备上，从而高效地利用多个硬件设备
* D：可以使用分区表来避免某些特殊的瓶颈，如：innodb的单个索引的互斥访问，ext3文件系统的inode锁竞争等
* E：如果需要，还可以备份和恢复独立的分区，这在非常大的数据集的场景下效果非常好
* F：优化查询，在where字句中包含分区列时，可以只使用必要的分区来提高查询效率，同时在涉及sum()和count()这类聚合函数的查询时，可以在每个分区上面并行处理，最终只需要汇总所有分区得到的结果。
---
## 分区本身也有一些限制：

* A：一个表最多只能有1024个分区（mysql5.6之后支持8192个分区）
* B：在mysql5.1中分区表达式必须是整数，或者是返回整数的表达式，在5.5之后，某些场景可以直接使用字符串列和日期类型列来进行分区（使用varchar字符串类型列时，一般还是字符串的日期作为分区）。
* C：如果分区字段中有主键或者唯一索引列，那么所有主键列和唯一索引列都必须包含进来，如果表中有主键或唯一索引，那么分区键必须是主键或唯一索引
* D：分区表中无法使用外键约束
* E：mysql数据库支持的分区类型为水平分区，并不支持垂直分区，因此，mysql数据库的分区中索引是局部分区索引，一个分区中既存放了数据又存放了索引，而全局分区是指的数据库放在各个分区中，但是所有的数据的索引放在另外一个对象中
* F：目前mysql不支持空间类型和临时表类型进行分区。不支持全文索引
---
## 子分区的建立需要注意以下几个问题：

* A：每个子分区的数量必须相同
* B：只要在一个分区表的任何分区上使用subpartition来明确定义任何子分区，就必须在所有分区上定义子分区，不能漏掉一些分区不进行子分区。
* C：每个subpartition子句必须包括子分区的一个名字
* D：子分区的名字必须是唯一的，不能在一张表中出现重名的子分区
* E：mysql数据库的分区总是把null当作比任何非null更小的值，这和数据库中处理null值的order by操作是一样的，升序排序时null总是在最前面，因此对于不同的分区类型，mysql数据库对于null的处理也各不相同。对于range分区，如果向分区列插入了null，则mysql数据库会将该值放入最左边的分区，注意，如果删除分区，分区下的所有内容都从磁盘中删掉了，null所在分区被删除，null值也就跟着被删除了。在list分区下要使用null，则必须显式地定义在分区的散列值中，否则插入null时会报错。hash和key分区对于null的处理方式和range,list分区不一样，任何分区函数都会将null返回为0.

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