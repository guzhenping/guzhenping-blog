# Hadoop学习指南（Kylin篇）
-------

## 一 Kylin介绍

## Kylin 安装与启动

需要hadoop的historyjobserver启动

## 度量计算

1. sum

2. min 
- max
- count
- count_distinct
- top_n
- raw
- extended_column
- percentile

## 实际问题

问题1：在同一个project里的Insight，可以看到所用cube的维度，但是不能共用，会报：No model found for rel

问题2：kylin不支持复杂的列，比如map,array类型。问题参见：
https://mail-archives.apache.org/mod_mbox/kylin-dev/201512.mbox/%3C565D565C.3060205@jd.com%3E

解决方法：创建该表的视图，剔除复杂列

问题3：kylin建model时，想做增量构建，在指定某列时间作为partition时，该列内容应该满足‘yyyy-MM-dd HH:mm:ss’的样子，不要使用unix_timestamp的类型。否则，构建后cube大小为零。

## Kylin的一些问题
关于Kylin Cube构建原理，落地到HBase的过程: [Apache Kylin Cube 构建原理](https://blog.bcmeng.com/post/kylin-cube.html)

关于Kylin SQL 语法：[SQL language](http://calcite.apache.org/docs/reference.html)

关于维度的聚合组中各个含义，请参考 
[https://kylin.apache.org/blog/2016/02/18/new-aggregation-group/](https://kylin.apache.org/blog/2016/02/18/new-aggregation-group/)

关于一次正常查询的运行原理：[Kylin进阶之路](https://zhuanlan.zhihu.com/p/30613434)

Kylin使用calcite做sql解析，可以参考calcite的语法文档：[https://calcite.apache.org/](https://calcite.apache.org/)

Kylin Mandatory Dimension(必要维度)：[【技术帖】Apache Kylin高级设置： 必要维度 （Mandatory Dimension）原理解析](https://mp.weixin.qq.com/s?__biz=MzAwODE3ODU5MA==&mid=2653077943&idx=1&sn=007d2ba345d0e25ec12807aa47f9913d&chksm=80a4bf46b7d33650465d33e20dac7edc09a7ad9308d77de6a501685c8ae00cba661c1d612074&scene=21#wechat_redirect)

Kylin Hierarchy Dimension(层级维度)：[【技术帖】Apache Kylin 高级设置：层级维度（Hierarchy Dimension）原理解析](https://mp.weixin.qq.com/s?__biz=MzAwODE3ODU5MA==&mid=2653077929&idx=1&sn=c76ed1fbb745945a077d9ca99f159a4d&chksm=80a4bf58b7d3364e0346ad9c433d4e32c57d45f41b361ae653c64c7fcebab21238793d2f66cb&scene=21#wechat_redirect)

Kylin Joint Dimension(联合维度)：[【技术帖】Apache Kylin 高级设置：联合维度（Joint Dimension）原理解析
](https://mp.weixin.qq.com/s?__biz=MzAwODE3ODU5MA==&mid=2653077926&idx=1&sn=a0037628bd102ec8e607d67204cbfa7c&chksm=80a4bf57b7d336419896c9e801a51f08ead2f7727d0d0ec0f9e3b7799ae3c302ebea54f93cc0&scene=21#wechat_redirect)

Kylin Aggregation Group(聚合组)：[【技术帖】Apache Kylin 高级设置：聚合组（Aggregation Group）原理解析](https://mp.weixin.qq.com/s?__biz=MzAwODE3ODU5MA==&mid=2653077921&idx=1&sn=89ae88bc63e71098166b74df7106c7bf&chksm=80a4bf50b7d3364692903aac3e901d09a516a8ff635e690e1e22b1d96abb4b2925c98cdace82&scene=21#wechat_redirect)

Kylin cube算法：[Apache Kylin的快速数据立方体算法——概述
](http://www.infoq.com/cn/articles/apache-kylin-algorithm)

Kylin cube介绍：[Kylin使用之创建Cube和高级设置](http://blog.csdn.net/yu616568/article/details/50570536)

脚本触发增量更新：[Kylin定时增量build](http://blog.csdn.net/aaronhadoop/article/details/52806486)

Cube优化案例：[Apache Kylin cube优化指南](http://www.jianshu.com/p/1e82e5dddae2)

Kylin比较详细的介绍：[Kylin对大数据量的多维分析](http://tech.meiyou.com/?p=97)

Kylin+superset 可视化方案案例：[Kylin初体验总结](http://zhuanlan.zhihu.com/p/26628057)


## 参考
很不错的博文：

 - [Apache Kylin 框架介绍](http://www.jianshu.com/p/6eadb77d091c)
 - [Apache Kylin在美团数十亿数据OLAP场景下的实践](http://www.infoq.com/cn/articles/kylin-apache-in-meituan-olap-scenarios-practice)