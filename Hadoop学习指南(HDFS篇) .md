# Hadoop学习指南(HDFS篇)
--------

## 前言
本篇介绍Hadoop的一些常用知识。要说和网上其他manual的区别，那就是这是笔者写的一套成体系的文档，不是随心所欲而作。

>Hadoop 
>
>DFS介绍
>
> Hadoop的分布式文件系统旨在：能够在跨大型集群中的机器上可靠地存储非常大的文件。它的灵感来自Google文件系统。Hadoop DFS将每个文件存储为一个块序列，文件中除最后一个块之外的所有块都具有相同的大小。属于文件的块将被复制以实现容错。块大小（block size）和复制因子(replication factor)可以按文件配置。HDFS中的文件是“一次写入”，并且在任何时间都有一个写入程序。

> Architecture

>像Hadoop Map/Reduce一样，HDFS遵循主/从架构。HDFS的安装包含一个Namenode,即一个主服务器，用于管理文件系统命名空间并控制客户端对文件的访问。此外，还有一些Datanodes,管理存储附加到他们运行的节点。Namenode进行文件系统命名空间操作，例如：通过RPC接口打开、关闭、重命名等文件和目录。它还确定块（block）到Datanodes的映射。Datanodes负责从文件系统客户端提供读取和写入请求，它们还根据Namenode的指令执行块创建、删除和复制。
>
>翻译自hadoop官网wiki

![](static/HDFS通讯.png)
>

## Hadoop常用命令
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

列出文件系统中各个文件由哪些块构成。

请将URI替换成自己想要查看的文件路径或文件夹路径。



## 参考资料

- [hadoop HDFS常用文件操作命令](https://segmentfault.com/a/1190000002672666)
- [HDFS 文件操作命令](http://book.51cto.com/art/201409/452359.htm)


