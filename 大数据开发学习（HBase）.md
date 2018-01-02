# Hadoop学习指南（HBase篇）
-------

## 一 HBase介绍

- HMaster：主要负责监控集群、管理RegionServers的负责均衡等，可以用主-备形式部署多个Master。
- HRegionServers：负责响应用户的I/O操作请求，客户端对HBase读写数据是与RegionServer交互。
- Zookeeper：负责选举Master的主节点；服务注册；保存RegionServers的状态等。可以使用系统内建的zookeeper，也可以使用独立的zookeeper，只需要在配置文件中调整即可。
- HDFS：真正的数据持久层，并非必须是HDFS文件系统，但搭配HDFS是最佳选择，也是目前应用最广泛的选择。







## 参考
作者：阿羅
链接：http://www.jianshu.com/p/6aeceb5d49cf
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。