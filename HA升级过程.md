#Hadoop 2.7.2 HA升级方案

## 前言
本次升级主要对Hadoop的core-site.xml和hdfs-site.xml文件进行修改，暂时不涉及其他配置。

这次升级过程，大致是三步：备份数据文件，修改配置文件，启动集群。如果升级中发现异常，启动回滚方案。


## 备份数据
### 备份配置文件

需要执行fabric脚本，在每台机器上进行备份。将core-site.xml和hdfs-site.xml分别cp为core-site.xml.ha.back和hdfs-site.xml.ha.back

以下是本次要修改的core-site.xml和hdfs-site.xml配置文件原内容。

- core-site.xml

```
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://sha2hdpn01:9000</value>
        </property>
        <property>
                <name>io.file.buffer.size</name>
                <value>131072</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/home/deploy/hadoopdata/tmp</value>
        </property>

        <property>
                <name>hadoop.proxyuser.deploy.groups</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.proxyuser.deploy.hosts</name>
                <value>*</value>
        </property>

		<property>
  			<name>hadoop.proxyuser.hue.hosts</name>
  			<value>*</value>
		</property>
		<property>
  			<name>hadoop.proxyuser.hue.groups</name>
  			<value>*</value>
		</property>

         <property>
                <name>fs.trash.interval</name>
                <value>10080</value>
        </property>

        <property>
               	<name>topology.script.file.name</name>
                <value>/usr/local/hadoop-default/etc/hadoop/topology.sh</value>
        </property>

        <property>
                <name>net.topology.script.file.name</name>
                <value>/usr/local/hadoop-default/etc/hadoop/topology.sh</value>
        </property>

</configuration>
```

- hdfs-site.xml

```
<configuration>
    <property>
           <name>dfs.namenode.secondary.http-address</name>
           <value>sha2hdpn02:50090</value>
    </property>
        <property>
           <name>dfs.namenode.http-address</name>
           <value>sha2hdpn01:50070</value>
    </property>
    <property>
           <name>dfs.namenode.name.dir</name>
           <value>/home/deploy/hadoopdata/namenode</value>
    </property>
    <property>
           <name>dfs.datanode.data.dir</name>
           <value>/home/deploy/hadoopdata/datanode/,/mnt/sdb/hadoopdata/datanode/</value>
    </property>
    <property>
           <name>dfs.replication</name>
           <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>/home/deploy/hadoopdata/checkpoint</value>
        <final>true</final>
    </property>
    <property>
        <name>dfs.namenode.checkpoint.edits.dir</name>
        <value>/home/deploy/hadoopdata/checkpoint</value>
    </property>
    <property>
        <name>dfs.namenode.checkpoint.period</name>
        <value>600</value>
    </property>
    <property>
        <name>dfs.datanode.du.reserved</name>
        <value>107374182400</value>
    </property>
    <property>
        <name>dfs.datanode.fsdataset.volume.choosing.policy</name>
        <value>org.apache.hadoop.hdfs.server.datanode.fsdataset.AvailableSpaceVolumeChoosingPolicy</value>
    </property>
    <property>
        <name>dfs.datanode.balance.bandwidthPerSec</name>
        <value>10485760</value>
    </property>
    <property>
        <name>dfs.hosts.exclude</name>
        <value>/usr/local/hadoop-default/etc/hadoop/exclude</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.datanode.balance.bandwidthPerSec</name>
        <value>52428800</value>
    </property>
    <property>
        <name>dfs.datanode.max.transfer.threads</name>
        <value>8192</value>
        <description>
            Specifies the maximum number of threads to use for transferring data
            in and out of the DN. default is 4096,###Modified###
        </description>
    </property>
    <property>
         <name>dfs.datanode.socket.write.timeout</name>
         <value>480000</value>
    </property>
   <property>
         <name>dfs.client.socket-timeout</name>
         <value>300000</value>
   </property>
   <property>
         <name>dfs.datanode.max.xcievers</name>
         <value>8192</value>
   </property>
   <property>
         <name>dfs.namenode.handler.count</name>
         <value>80</value>
   </property>
</configuration>
```

### 备份namenode数据

需要关闭集群后，在sha2hdpn01上备份namenode的元数据,位置：`/home/deploy/hadoopdata`,大小：25G。

### 备份secondary namenode的数据
需要关闭集群后，在sha2hdpn02上备份snn的元数据，位置：`/home/deploy/hadoopdata`, 大小：774G。


## 修改配置文件

### 修改core-stie.xml

```
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hacluster</value>
        </property>
        <property>
                <name>io.file.buffer.size</name>
                <value>131072</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/home/deploy/hadoopdata/tmp</value>
        </property>
        <property>
            <name>ha.zookeeper.quorum</name>
            <value>sha2hb01:2181,sha2hb02:2181,sha2hb03:2181</value>
        </property>
		 
        <property>
                <name>hadoop.proxyuser.deploy.groups</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.proxyuser.deploy.hosts</name>
                <value>*</value>
        </property>

		<property>
  			<name>hadoop.proxyuser.hue.hosts</name>
  			<value>*</value>
		</property>
		<property>
  			<name>hadoop.proxyuser.hue.groups</name>
  			<value>*</value>
		</property>

         <property>
                <name>fs.trash.interval</name>
                <value>10080</value>
        </property>

        <property>
               	<name>topology.script.file.name</name>
                <value>/usr/local/hadoop-default/etc/hadoop/topology.sh</value>
        </property>

        <property>
                <name>net.topology.script.file.name</name>
                <value>/usr/local/hadoop-default/etc/hadoop/topology.sh</value>
        </property>

</configuration>
```

### 修改hdfs-site.xml

先删除secondary namenode配置，在添加和ha配置相关的内容。


```
<configuration>
    
    <property>
           <name>dfs.namenode.name.dir</name>
           <value>/home/deploy/hadoopdata/namenode</value>
    </property>
    <property>
           <name>dfs.datanode.data.dir</name>
           <value>/home/deploy/hadoopdata/datanode/,/mnt/sdb/hadoopdata/datanode/</value>
    </property>
    <property>
           <name>dfs.replication</name>
           <value>2</value>
    </property>
    
    <property>
        <name>dfs.nameservices</name>
        <value>hacluster</value>
    </property>
    <property>
        <name>dfs.ha.namenodes.hacluster</name>
        <value>nn1,nn2</value>
    </property>
    
    <property>
        <name>dfs.namenode.rpc-address.hacluster.nn1</name>
        <value>sha2hdpn01:9000</value>
    </property>
    
    <property>
        <name>dfs.namenode.rpc-address.hacluster.nn2</name>
        <value>sha2hdpn02:9000</value>
    </property>

     <property>
        <name>dfs.namenode.http-address.hacluster.nn1</name>
        <value>sha2hdpn01:50070</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.hacluster.nn2</name>
        <value>sha2hdpn02:50070</value>
    </property>
    
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://sha2hdpw46:8485; sha2hdpw47:8485;sha2hdpw48:8485/hacluster</value>
    </property>
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/home/deploy/hadoopdata/journaldata</value>
    </property>
    
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.client.failover.proxy.provider.hacluster</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>

    <property>
      <name>dfs.ha.fencing.methods</name>
      <value>
            sshfence
            shell(/bin/true)
      </value>
    </property>
    <property>
      <name>dfs.ha.fencing.ssh.private-key-files</name>
      <value>/home/deploy/.ssh/id_rsa</value>
    </property>
    <property>
        <name>dfs.ha.fencing.ssh.connect-timeout</name>
        <value>30000</value>
    </property>
    
    <property>
        <name>dfs.datanode.du.reserved</name>
        <value>107374182400</value>
    </property>
    <property>
        <name>dfs.datanode.fsdataset.volume.choosing.policy</name>
        <value>org.apache.hadoop.hdfs.server.datanode.fsdataset.AvailableSpaceVolumeChoosingPolicy</value>
    </property>
    <property>
        <name>dfs.datanode.balance.bandwidthPerSec</name>
        <value>10485760</value>
    </property>
    <property>
        <name>dfs.hosts.exclude</name>
        <value>/usr/local/hadoop-default/etc/hadoop/exclude</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.datanode.balance.bandwidthPerSec</name>
        <value>52428800</value>
    </property>
    <property>
        <name>dfs.datanode.max.transfer.threads</name>
        <value>8192</value>
        <description>
            Specifies the maximum number of threads to use for transferring data
            in and out of the DN. default is 4096,###Modified###
        </description>
    </property>
    <property>
         <name>dfs.datanode.socket.write.timeout</name>
         <value>480000</value>
    </property>
   <property>
         <name>dfs.client.socket-timeout</name>
         <value>300000</value>
   </property>
   <property>
         <name>dfs.datanode.max.xcievers</name>
         <value>8192</value>
   </property>
   <property>
         <name>dfs.namenode.handler.count</name>
         <value>80</value>
   </property>
</configuration>
```

## 启动集群
先启动zookeeper集群，并确定状态；

再启动journalnode集群，用于nn间数据同步（注意文件夹存储的位置和权限）；

在主节点上启动namenode；

在副节点上，同步namenode的元数据：`hdfs namenode -bootstandby`（metadata为7.3G, 注意磁盘空间及时长）；

在副节点上启动namenode;

在任意节点上，启动dfs: `sh start-dfs.sh`;

启动其他的。

## 回滚方案
此处升级，主要是对core-site.xml和hdfs-site.xml文件进行修改，所以回滚方案的主要逻辑就是恢复这两份文件。

利用fabric写脚本，先每台机器上备份（cp命令），如需回滚，先关闭集群，再把该文件替换回来（mv命令）即可。

附脚本rollback_hadoop.py：

```python
from fabric.api import run,sudo,roles,env,cd,execute

env.roledefs = {
    'all_node': ['sha2hdpn01','sha2hdpn02','sha2hdpw01','sha2hdpw02','sha2hdpw03','sha2hdpw04','sha2hdpw05','sha2hdpw06','sha2hdpw07','sha2hdpw08','sha2hdpw09','sha2hdpw10','sha2hdpw11','sha2hdpw12','sha2hdpw13','sha2hdpw14','sha2hdpw15','sha2hdpw16','sha2hdpw17','sha2hdpw18','sha2hdpw19','sha2hdpw20','sha2hdpw21','sha2hdpw22','sha2hdpw23','sha2hdpw24','sha2hdpw25','sha2hdpw26','sha2hdpw27','sha2hdpw28','sha2hdpw29','sha2hdpw30','sha2hdpw31','sha2hdpw32','sha2hdpw33','sha2hdpw34','sha2hdpw35','sha2hdpw36','sha2hdpw37','sha2hdpw38','sha2hdpw39','sha2hdpw40','sha2hdpw41','sha2hdpw42','sha2hdpw43','sha2hdpw44','sha2hdpw45','sha2hdpw46','sha2hdpw47','sha2hdpw48'],
    'namenode': ['sha2hdpn01'],
    'test_node': ['sha2hdpw48']
}
env.user = 'deploy'
env.password = 'XXXXXX' # yourself
# env.shell = '/bin/sh -c'

@roles('all_node')
def showfile():
    # run('ll /usr/local/hadoop-default/etc/hadoop')i
    with cd('/usr/local/hadoop-default/etc/hadoop'):
        run("ls core-site.xml hdfs-site.xml")

@roles('namenode')
def start_hadoop():
    print('start hadoop cluster...')
    # run('sh /usr/local/hadoop-default/sbin/start-all.sh')
    print('done...')

@roles('all_node')
#@roles('test_node')
def backup():
    print('start backup...')
    with cd('/usr/local/hadoop-default/etc/hadoop'):
        run('cp core-site.xml core-site.xml.ha.back')
        run('cp hdfs-site.xml hdfs-site.xml.ha.back')
    print('done...')

@roles('all_node')
#@roles('test_node')
def rollback():
    print('start roolback...')
    with cd('/usr/local/hadoop-default/etc/hadoop'):
        run('mv core-site.xml.ha.back core-site.xml')
        run('mv hdfs-site.xml.ha.back hdfs-site.xml')
    print('done...')

def deploy():
    execute(showfile)

def run_backup():
    execute(backup)

def run_rollback():
    execute(rollback)
    execute(start_hadoop)
```


## 测试方案
重跑airflow中金融/流量的报表任务，用于提交job，查看是否能够完整跑完。这些任务中有使用了hdfs及yarn的操作，如果成功，说明hadoop ha集群可以用。

## 结语
烧香拜佛，希望成功！

