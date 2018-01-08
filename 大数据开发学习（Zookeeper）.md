# 大数据开发学习（ZooKeeper）
-------

## 一 前言

ZooKeeper有三种配置方式：单节点、伪集群和集群。本文以集群配置方式为例。

环境介绍：

- CentOS 6.5 64位
- Java 1.8
- ZooKeeper 3.4.9

## 二 安装
下载：`wget http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz`
如有版本需求，修改上述的版本号（3.4.9）即可

解压到当前目录：`tar -zxvf zookeeper-3.4.9.tar.gz`


配置过程，在网上有大量的教程，可以参照[《在 CentOS7 上安装 Zookeeper-3.4.9 服务》](http://www.linuxidc.com/Linux/2016-09/135052.htm)。

提示，在很多配置过程中，不建议新手使用hostname来配置，尽管你很熟悉hosts文件，也不建议。请使用IP来完成相关的节点配置。


## 三 使用
运行：

- `zkServer.sh start-foreground`
- `zkServer.sh start`


## 参数详解
[zookeeper的配置参数详解（zoo.cfg）](http://www.cnblogs.com/xiohao/p/5541093.html)

## 六 参考资料

- [zookeeper 学习（一）](https://lanjingling.github.io/2016/02/21/zookeeper-study1/)
- [zookeeper工作原理、安装配置、工具命令简介](http://www.cnblogs.com/kunpengit/p/4045334.html)