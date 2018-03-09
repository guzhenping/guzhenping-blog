# Hadoop学习指南（集群运维篇）
-------

## 前言
本篇介绍Hadoop的一些常用知识。要说和网上其他manual的区别，那就是这是笔者写的一套成体系的文档，不是随心所欲而作。


## 常用HDFS命令

- hadoop fs -ls URI
- hadoop fs -du -h URI
- hadoop fs -cat URI [文件较大，hadoop fs -cat xxxx | head]
- hadoop fs -put URI
- hadoop fs -get URI
- hadoop fs -rmr URI
- hadoop fs -stat %b,%o,%n,%r,%y URI (%b：文件大小, %o：Block 大小, %n：文件名, %r：副本个数, %y 或%Y：最后一次修改日期和时间)
- hadoop fs -tail [-f] URI
- hdfs dfsadmin -report
- hadoop fs -appendToFile URI1[,URI2,...] URI（hadoop fs -appendToFile helloCopy1.txt helloCopy2.txt /user/tmp/hello.txt）
- hadoop fsck / -files -blocks

## 关键的clusterID
这是hadoop集群的id号，只有拥有相同id的各个节点才能加入的这个集群中来。

大致是在：hdfs namenode -format命令之后生成这个id。

很多情况下，你的集群如果可能拥有不同的id号哦。就比如：

启动datanode是遇到： “DataNode: Initialization failed for Block pool”

此时应当查看：
cat /home/deploy/hadoop-2.7.1/hdfs/name/current/VERSION

cat /home/deploy/hadoop-2.7.1/hdfs/data/current/VERSION

一个是datanode数据的文件夹，一个是namenode数据的文件夹。

```
#Fri Feb 17 15:48:44 CST 2017
namespaceID=74707331
clusterID=CID-e3e7c80e-3099-461d-9fa9-404f4910def5
cTime=0
storageType=NAME_NODE
blockpoolID=BP-1729560578-192.168.20.2-1487317724421
layoutVersion=-63
```

```
#Thu Dec 22 09:57:13 CST 2016
storageID=DS-63997596-6d60-46e8-a08c-94426208f9d9
clusterID=CID-076b4e8a-9ed9-47e9-b6e0-4c16440c33e8
cTime=0
datanodeUuid=3ae0642e-2d72-4b45-ba63-de5e6ca1c7c7
storageType=DATA_NODE
layoutVersion=-56
```

## 添加删除节点
[Hadoop添加删除节点](https://my.oschina.net/MrMichael/blog/291802)



## 重启丢失节点

### 子节点DataNode丢失

`sbin/hadoop-daemon.sh start datanode`

### 子节点NodeManager丢失

`sbin/yarn-daemon.sh start nodemanager`

### 主节点丢失
`sbin/start-all.sh`

or

`sbin/hadoop-daemon.sh start namenode`

`sbin/hadoop-daemon.sh start secondarynamenode`

`sbin/yarn-daemon.sh start resourcemanager`

## 配置文件出错
管理hadoop集群，会经常遇到配置文件的相关问题。这里举一个例子，比如yarn的nodemanager起不来的问题。

yarn的相关配置文件有两个：yarn-site.xml和yarn-env.sh

在yarn-site.xml文件：

```
<property>
     <name>yarn.nodemanager.resource.memory-mb</name>
     <value>1024</value>
</property>
```
在yarn-env.sh文件：

```
JAVA_HEAP_MAX=-Xmx1024m
```

应该保证yarn-site.xml中的memory-mb数值比yarn-env.sh中JAVA_HEAP_MAX的数值小。yarn-site.xml的配置是要求nodemanager启动的最少内存，低于该值无法启动。实际启动时，使用yarn-env.sh中的配置。修改比如：`JAVA_HEAP_MAX=-Xmx2048m`


## no xxx to stop

hadoop会经常有这个问题，大概就是没有找到该进程的PID文件，所以报错。

具体参见连接：
[解决关闭Hadoop时no namenode to stop异常](http://blog.csdn.net/gyqjn/article/details/50805472)

每次启动hadoop（`./start-all.sh`）时，PID文件被生成，存储进程号。关闭时，PID文件被删除。

在hadoop2.7.1版本中，关于HADOOP_PID_DIR(文件路径：../etc/hadoop/hadoop-env.sh)的描述是这样的：

```
# The directory where pid files are stored. /tmp by default.
# NOTE: this should be set to a directory that can only be written to by
#       the user that will run the hadoop daemons.  Otherwise there is the
#       potential for a symlink attack.
export HADOOP_PID_DIR=${HADOOP_PID_DIR}
export HADOOP_SECURE_DN_PID_DIR=${HADOOP_PID_DIR}
```

最好将PID文件放在只写目录中。

## 关于mapred-site.xml配置

参见blog:[《如何给运行在Yarn的MapReduce作业配置内存》](https://www.iteblog.com/archives/1945)



## 参考资料

-[如何给运行在Yarn的MapReduce作业配置内存](https://www.iteblog.com/archives/1945)
- [hadoop HDFS常用文件操作命令](https://segmentfault.com/a/1190000002672666)
- [HDFS 文件操作命令](http://book.51cto.com/art/201409/452359.htm)