# HA成功升级的总结

## 前言
从升级到现在，一共过了1个半月，到昨天（20170511）总算踏实了。踩的那些坑，真的教会了笔者咋做人。升了，笔者并不后悔。当然脸被打了这么多下，也高兴不起来。

升级过程的所有问题，发生在当事人无法注意到、无法理解的地方。然而，只要时间充足，精力充沛，严谨严格，肯定能克服。

必须申明：本文是基于个体实际经验所写，难免片面，可能不具备参考价值。其次，本文是对一个已经存在数年有大数量的集群所做的一些升级，情况较为特殊。


## 升级准备
这里就不多说了，参见[《Hadoop Namenode HA升级》](https://www.tapd.cn/20096511/markdown_wikis/#1120096511001000954)。

总结一句，准备和成功概率成正比，有付出，才有收获。



## 问题1：异构的配置文件

对于hadoop集群来说，很多人都觉得hadoop配置文件是一样的。错了，真的错了。hadoop集群支持异构模式。所以，配置可以不同。

在本次升级过程中，配置上，遇到的问题：个别机器的盘符不同于大部分机器，个别机器的datanode数据所在盘没有填写在`<name>dfs.datanode.data.dir</name>`下。

对于linux盘符不太熟悉的观众请戳：[《Linux硬盘盘符分配》](http://ilinuxkernel.com/?p=958)

例如，有以下两种（及以上）配置：

```
<property>
       <name>dfs.datanode.data.dir</name>
       <value>/home/deploy/hadoopdata/datanode/,/mnt/sdb/hadoopdata/datanode/</value>
</property>
    
```

```
<property>
       <name>dfs.datanode.data.dir</name>
       <value>/home/deploy/hadoopdata/datanode/,/mnt/sdc/hadoopdata/datanode/,/mnt/sdd/hadoopdata/datanode/</value>
</property>
```

这种错误的配置，会导致namenode在启动过程中无法找到block，处于安全模式无法退出。对于集群停机不能超过5小时的公司/团体来说，只能强制立刻安全模式。此时，将会产生坏块。

对于hadoop集群来说，必须清除坏块。换句话说，因为配置的失误，会导致datanode丢数据。这种丢法，和没有挂上的那个/些盘有关，而且一丢就是丢一个盘，后果严重。

当然hadoop的块备份是大于等于2，如果只是一个盘，对于集群来说就相当于没有丢。反之，则是随机丢失块数据。


## 问题2：Hadoop堆内存

先说结论：对于HA来说，两个namenode（active/standby）的堆内存应该要比hadoop集群metadata大至少1倍。这个结论并没有理论根据，但确实实践所得的最可靠数据。假设一个运行5年的集群有20G metadata数据，则namenode(a/s)需要40G以上。

为啥这么说？因为，namenode主备自动切换时(即主namenode异常，备namenode启动)，standby namenode需要将metadata读到进程堆内存中。堆内存不住，NameNode进程会报GC堆已满。GC堆相关问题请戳：[《深入理解JVM—JVM内存模型》
](http://www.cnblogs.com/dingyingsi/p/3760447.html)

理解主备热切，大致有两个要点，本文不多说。只点一下，第一，单节点Namenode启动过程发生了哪些事情；第二，HA集群namenode启动中发生了哪些事情。主要就是围绕FSImage和EditsLog展开。请戳下方链接：

- [《NameNode启动过程详细剖析 NameNode中几个关键的数据结构 FSImage》](http://blog.csdn.net/cnhk1225/article/details/50786785)
- [《Hadoop-2.X HA模式下的FSImage和EditsLog合并过程》](http://blog.csdn.net/dabokele/article/details/51686257)

对于拥有20G左右metadata的集群，standby需要10-20分钟内就可以启动(堆内存够大)。如果热切时，standby namenode的内存就16G,那么半个小时差不多只能走到25%左右。

堆内存大点很易理解，数据多嘛。而且，网上有很多hadoop在提交作业时报jvm堆内存不足的问题，可以参考。

笔者的小伙伴在解决standby namenode启动过慢这个问题时，通过修改安装目录的`../etc/hadoop/hadoop-env.sh`文件中的HADOOP_HEAPSIZE这个参数。一开始,该参数同集群metadata数据大小无异，第二次改成约1.5倍，第三次改成约2倍。

HADOOP_HEAPSIZE的值会成为本地（不是每台机器共用的，可以不同）的JVM的堆大小。本地的各个Java守护进程都会共享这个堆，此时进程NameNoe能满足快速启动的条件。虽然会有其他java进程也用，但是HA模式下的namenode没有过多的java进程（其实就是DFSZKFailoverController）。


## 问题3：NameNode RPC调用方式
未升级前，连接hadoop集群通过`<name>fs.defaultFS</name>`下的hdfs://IP：PORT来连。升级后，需要采用命名空间的方式：

```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://mainhadoop</value>
</property>
```

即：hdfs://命名空间。连接上命名空间后，zookeeper集群会自动分配程序连接到当前active的namenode。

对此，受影响较大的是一些执行脚本和已存在的hive表。执行脚本一般写死，只能一个一个修改。

对于hive表，需要进入到元数据所在数据库，修改数据Location指向。如果hive是以mysql作元数据存储，则需连上mysql，修改SDS和DBS两张表的数据。将“hdfs://ip:port/XXXXXXXXXX”改成新的hadoop命名空间。




