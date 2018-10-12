# 大数据开发学习（Redis）
-------

## 一 前言
经常使用redis, 特地进行总结。


## 二 基础

#### 安装
下载安装包，或者：

```
$ wget http://download.redis.io/releases/redis-3.2.0.tar.gz
$ tar xzf redis-3.2.0.tar.gz
$ cd redis-3.2.0
$ make
```
就是make命令~中间过程，报什么错搞定什么即可。


Ubuntu下比较简单，就是apt-get install redis-server

默认安装位置： 利用whereis redis 去找redis.conf ，需要修改是否后台运行daemonize （为yes）等属性

完成后，自动启动，可用命令：ps -aux| grep redis

redis是依赖，因为我们用python，不得不装一个wrapper——redis-py:
`sudo pip install redis`


 
#### 启动
要让Redis-server在后台运行！

1.简单的启动：

进到redis目录下的src文件夹下，输入：
redis-server

2.后台启动：

先去找redis.conf文件，修改daemonize属性，从no变为yes
进到redis目录下的src文件夹下，
输入：
redis-server &


3.通过配置文件启动：

需要配置启动文件，在Redis工程目录下有个redis.conf文件，修改：

```
#修改daemonize为yes，即默认以后台程序方式运行（还记得前面手动使用&号强制后台运行吗）。
daemonize yes

#可修改默认监听端口，别改了，万一你忘了
port 6379

#修改生成默认日志文件位置
logfile  "/home/futeng/logs/redis.log"

#配置持久化文件存放位置
dir /home/futeng/data/redisData
```
配置完以后，启动：
还是 src目录下，`redis-server ./redis.conf`

if 改了端口，使用redis-cli命令连接时，需要带上端口，比如：
redis-cli -p xxxx  [再次强调：仍在src目录下]

4.使用redis启动脚本，设置开机自启
在生产环境中使用这种方式。

5.启动无密码验证的

`redis-server --protected-mode no`

设置连接数

`redis-server --protected-mode no --maxclients 100000`


#### 客户端

还是进到redis目录下的src文件夹下，输入：
redis-cli

在弹出的交互界面测试下：

```
redis> set foo bar
OK
redis> get foo
"bar"
```


