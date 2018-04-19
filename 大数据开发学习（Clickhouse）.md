
## 前言
1年前，用过Greenplum，这是第一次接触MPP结构的OLAP系统。今天市面上常见的MPP架构工具，还有Clickhouse和Palo等。(当然，SQL on Hadoop体系的Presto和Impala也算是MPP结构，只是数据存储方面没有自己的东西，都是依赖hdfs,mysql等。)

工作需要，对Clickhouse进行学习。

## 安装与启动

Ubuntu上比较好安装，但是一般公司用的服务器都是Centos。这里只讨论在Centos 7的安装方式，centos 6请在Altinity公司提供的下载界面中自行寻找。

### 安装命令

```
如果是装54362版本的包，其余所有依赖都需要是一致的。

准备源依赖，由Altinity公司提供：
curl -s https://packagecloud.io/install/repositories/Altinity/clickhouse/script.rpm.sh | sudo bash

server-common:
sudo yum install clickhouse-server-common-1.1.54362-1.el7.x86_64

server:
sudo yum install clickhouse-server-1.1.54362-1.el7.x86_64

client：
sudo yum install clickhouse-client-1.1.54362-1.el7.x86_64

```

关于Altinity公司的其他版本，可访问[这里下载](https://packagecloud.io/Altinity/clickhouse)。

以上安装如有疑问，可以使用下方安装方式：

- [https://github.com/red-soft-ru/clickhouse-rpm](https://github.com/red-soft-ru/clickhouse-rpm)


### 启动

```
server端：
sudo /etc/rc.d/init.d/clickhouse-server start

client端：
clickhouse-client

```

## 使用经验

### 常用SQL

```
-- 查看集群情况
select * from system.clusters;

-- 查看分区情况
select 
	partition,
	name,
	rows
from system.parts;

```


### MergeTree
选择engine时，尽量用merge tree.MergeTree引擎支持以主键和日期作为索引，提供实时更新数据的可能性。这是Clickhouse中最先进的表引擎。

数据表将数据分割为小的索引块进行处理。每个索引块之间依照主键排序。每个索引块记录了指定的开始日期和结束日期。插入数据时，MergeTree会对数据进行排序，以保证存储在索引块中的数据有序。当写入新数据时，会放在新的文件夹下，索引块之间的合并过程会在系统后台定期自动执行。MergeTree引擎会选择几个相邻的索引块进行合并，然后对二者合并排序。

![](static/clickhouse/mergetree1.png)

![](static/clickhouse/mergetree2.png)

![](static/clickhouse/mergetree3.png)

![](static/clickhouse/mergetree4.png)

![](static/clickhouse/mergetree5.png)

![](static/clickhouse/mergetree6.png)

![](static/clickhouse/mergetree7.png)



### Distributed
相当于数据库的视图，并不存储数据，而是用来做分布式的写入和查询，与其他引擎结合使用。

一种集群拓扑结构：
![](static/clickhouse/distributed.jpg)


## 实际问题

1. 关键字大小写敏感，对强转支持不太好。
2. 特性上不支持事务，不支持update/delete。
3. 关于分布式
![](static/clickhouse/question_clickhouse1.png)
4. 在分布式表上执行count()时，发现结果不一致。

暂无解决办法。问题:[https://github.com/yandex/ClickHouse/issues/1443](https://github.com/yandex/ClickHouse/issues/1443)

5. 查询时，内存超出限制：

```
Progress: 4.84 million rows, 42.70 MB (45.34 million rows/s., 399.59 MB/s.)  87%

Received exception from server:
Code: 241. DB::Exception: Received from localhost:9000, ::1. DB::Exception: Memory limit (for query) exceeded: would use 12.15 GiB (attempt to allocate chunk of 4294967296 bytes), maximum: 9.31 GiB.

0 rows in set. Elapsed: 48.023 sec. Processed 4.84 million rows, 42.70 MB (100.89 thousand rows/s., 889.21 KB/s.)

Progress: 4.89 million rows, 43.81 MB (23.23 thousand rows/s., 208.09 KB/s.)  88%

Received exception from server:
Code: 241. DB::Exception: Received from localhost:9000, ::1. DB::Exception: Memory limit (for query) exceeded: would use 80.15 GiB (attempt to allocate chunk of 17179869184 bytes), maximum: 74.51 GiB.

3668208133 rows in set. Elapsed: 235.536 sec. Processed 4.89 million rows, 43.81 MB (20.77 thousand rows/s., 186.00 KB/s.)

Received exception from server (version 1.1.54362):
Code: 241. DB::Exception: Received from localhost:9000, ::1. DB::Exception: Memory limit (for query) exceeded: would use 96.14 GiB (attempt to allocate chunk of 17179869184 bytes), maximum: 93.13 GiB.

8816716667 rows in set. Elapsed: 428.543 sec. Processed 4.89 million rows, 43.81 MB (11.41 thousand rows/s., 102.23 KB/s.)

```
当处理的数据集超过设定的阈值以后，会触发限制，并返回已经处理好的结果。

解决办法：
修改max\_memory\_usage的值（user.xml）

不同的节点可以处理设置不同。在高并发分流的时候，尽量把query分摊到各个机器上，否则会将某一节点资源耗尽。

这个问题导致Clickhouse的高并发特性很差。在前端暴露端口时，不能单独暴露一个host,需要做成有命名空间的方式，或者有个query平衡器的机制。


## 参考资料


### 官方文档
[What is ClickHouse](https://clickhouse.yandex/docs/en/single/#introduction)

### 配置文件

[ClickHouse相关配置剖析](https://kuaibao.qq.com/s/20180409G06IIM00?refer=spider)

[ClickHouse的分布式引擎](http://note.abeffect.com/note/articles/2017/12/18/1513590469620.html)

### 用户权限

[ClickHouse 用户名密码设置](https://www.jianshu.com/p/e339336e7bb9)

[ClickHouse之访问权限控制](http://www.cnblogs.com/gomysql/p/6708796.html)

### 引擎介绍

[ClickHouse MergeTree引擎介绍](https://www.jianshu.com/p/48dbf2db2765)

[ClickHouse Distribute 引擎深度解读](http://www.clickhouse.com.cn/topic/5a3e768d2141c2917483557e)

[实时大数据分析引擎ClickHouse介绍](http://www.linkedkeeper.com/detail/blog.action?bid=1117)

### 数据同步
kafka->Clickhouse：[Hangout with ClickHouse](http://jackpgao.github.io/2017/12/27/ClickHouse-with-Hangout/)

mysql->Clickhouse：[使用ClickHouse一键接管MySQL数据分析](http://jackpgao.github.io/2018/02/04/ClickHouse-Use-MySQL-Data/)


### 杂文
奶clickhouse的文章：[ClickHouse Beijing Meetup-数据分析领域的黑马-ClickHouse-新浪-高鹏](https://zhuanlan.zhihu.com/p/33371816)