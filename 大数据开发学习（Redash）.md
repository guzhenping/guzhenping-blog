# Redash开发指南
-------

## 一 Redash介绍




## 二 测试环境

#### 官方提供的测试环境启动方式
redash开发文档 ： https://redash.io/help-onpremise/


#### 自主搭建
安装虚拟环境管理工具anaconda, 创建虚拟环境redash:`conda create -n redash python=2.7`,激活该环境:`source activate redash`。

创建虚拟环境（也可以不创建），激活后。利用pip安装redash所需要的包。

```
pip install -r requirements.txt      # 程序基础依赖
pip install -r requirements_all_ds.txt    # 所有数据库的依赖
```

在上述安装过程中大概率会出现安装错误(遇到过10多回)。请将出错的包单独安装。

安装npm，在主目录执行：

```
npm install    # 安装node模块
npm run build    # 编译前端的东西
```

启动服务器: 
```
bin/run ./manage.py runserver --debugger --reload
```

启动celery的woker和调度器:
```
./bin/run celery worker --app=redash.worker --beat -Qscheduled_queries,queries,celery -c2
```

也可以分开启动woker和调度器，调度器启动:

```
bin/run celery worker --app=redash.worker -c4 -Qscheduled_queries --maxtasksperchild=10 -Ofair
```

worker启动:
```
bin/run celery worker --app=redash.worker --beat -c8 -Qqueries,celery --maxtasksperchild=10 -Ofair
```

**根据机器机器实际情况，调节-c后的参数，用来指定启动多少个进程数量。**

#### 启动方式
除了上面的启动方式，还推荐使用guncoin的启动方式。

启动服务器：

```
# 前台启动
gunicorn -b 127.0.0.1:5000 --name redash -w 4 --max-requests 1000 redash.wsgi:app

# 后台启动
nohup /home/hadoop/anaconda3/envs/redash/bin/python /home/hadoop/anaconda3/envs/redash/bin/gunicorn -b 127.0.0.1:5000 --name redash -w 4 --max-requests 1000 redash.wsgi:app >> redash.log &
```

没有gunicorn命令的，需要装Python包。

启动调度器进程：

```
# 前台启动
bin/run celery worker --app=redash.worker -c4 -Qscheduled_queries --maxtasksperchild=10 -Ofair

# 后台启动
nohup /home/hadoop/anaconda3/envs/redash/bin/celery worker --app=redash.worker -c4 -Qscheduled_queries --maxtasksperchild=10 -Ofair >> schedular.log &
```

启动worker进程：

```
# 前台启动
bin/run celery worker --app=redash.worker --beat -c8 -Qqueries,celery --maxtasksperchild=10 -Ofair  

# 后台启动
nohup /home/hadoop/anaconda3/envs/redash/bin/celery worker --app=redash.worker --beat -c8 -Qqueries,celery --maxtasksperchild=10 -Ofair  >> worker.log &
```

也可以用nginx做下代理，配置文件：

```
worker_processes  4;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  3600;

    upstream rd_servers {
         server 127.0.0.1:5000;
    }
    server {
          server_tokens off;
          listen 5123 default;
          access_log /var/log/nginx/rd.access.log;
          gzip on;
          gzip_types *;
          gzip_proxied any;
          location / {
              proxy_set_header Host $http_host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              proxy_set_header X-Forwarded-Proto $scheme;
              proxy_pass       http://rd_servers;
          }
    }
}
```
启动NGINX:

```
cd /usr/local/nginx/sbin
sudo ./nginx
```

## 三 Redash VS Superset
关于Redash：https://redash.io/；与Superset的区别与联系：https://www.zhihu.com/question/60369195/answer/258298127。


## Bug
hive 读 schema
show columns in table , 无法处理中文comment,
desc table , 无法处理没有字段的表，
报错：Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. java.lang.ClassNotFoundException Class org.apache.hive.hcatalog.data.JsonSerDe not found


## 参考
