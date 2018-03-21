# 大数据开发学习（Spark）
-------

## 一 前言
平时接触了一些spark的应用场景，持续保持学习。

## 二 安装
一般都用on yarn的方式

## 三 实际问题

Kylin在采用spark作为计算引擎时，需要进行参数配置。以下答案给了很好的启发性，特地记录：

>Note that yarn.nodemanager.resource.memory-mb is total memory that a single NodeManager can allocate across all containers on one node.
In your case, since yarn.nodemanager.resource.memory-mb = 12G, if you add up the memory allocated to all YARN containers on any single node, it cannot exceed 12G.
You have requested 11G (-executor-memory 11G) for each Spark Executor container. Though 11G is less than 12G, this still won't work. Why ?
* Because you have to account for spark.yarn.executor.memoryOverhead, which is min(executorMemory * 0.10, 384) (by default, unless you override it).
So, following math must hold true:
spark.executor.memory + spark.yarn.executor.memoryOverhead <= yarn.nodemanager.resource.memory-mb
See: https://spark.apache.org/docs/latest/running-on-yarn.html for latest documentation on spark.yarn.executor.memoryOverhead
Moreover, spark.executor.instances is merely a request. Spark ApplicationMaster for your application will make a request to YARN ResourceManager for number of containers = spark.executor.instances. Request will be granted by ResourceManager on NodeManager node based on:
* Resource availability on the node. YARN scheduling has its own nuances - this is a good primer on how YARN FairScheduler works.
* Whether yarn.nodemanager.resource.memory-mb threshold has not been exceeded on the node:
    * (number of spark containers running on the node * (spark.executor.memory + spark.yarn.executor.memoryOverhead)) <= yarn.nodemanager.resource.memory-mb*
If the request is not granted, request will be queued and granted when above conditions are met

>
>from:
>https://stackoverflow.com/questions/29940711/apache-spark-setting-executor-instances-does-not-change-the-executors


## 四 参考资料

