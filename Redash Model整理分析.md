# Redash 源码分析


## 前言
掌握Redash执行原理，对于深度的二次开发至关重要。



## query功能
SQL查询是Redash的核心功能之一。通常情况下，用户在前端会生成如下参数：

- query.data_source：下列列表中的数据源，必须选
- parameter_values：用户自定义的参数值，可无
- query.query_text：用户自定义的SQL文本，必须写
- query.id：前端自动生成的查询id，系统生成

在上述条件具备后，将会调用Redash API： `/api/queries/`进行任务提交。

所有任务将会被redash.handlers.query_result.run_query()方法接受处理。此处将进行参数校验，语法解析，任务传输等方面的处理。

多数的查询场景，先将SQL文本hash后，去缓存队列和后台数据库（Redash的Postgre）进行寻找，找不到再发给celery让worker执行该SQL。

负责具体执行：redash.models.QueryResult()类实例 和redash.tasks.queries.enqueue_query()方法。前者：负责查现有的结果；后者：立即执行相关SQL语句。

在enqueue_query()方法里，先连Redis,将任务加入队列，持续监听查询结果。

核心代码如下：

```
    while try_count < 5:
        try_count += 1

        pipe = redis_connection.pipeline()
        try:
            pipe.watch(_job_lock_id(query_hash, data_source.id))
            job_id = pipe.get(_job_lock_id(query_hash, data_source.id))
            if job_id:
                logging.info("[%s] Found existing job: %s", query_hash, job_id)

                job = QueryTask(job_id=job_id)

                if job.ready():
                    logging.info("[%s] job found is ready (%s), removing lock", query_hash, job.celery_status)
                    redis_connection.delete(_job_lock_id(query_hash, data_source.id))
                    job = None

            if not job:
                pipe.multi()

                time_limit = None

                if scheduled_query:
                    queue_name = data_source.scheduled_queue_name
                    scheduled_query_id = scheduled_query.id
                else:
                    queue_name = data_source.queue_name
                    scheduled_query_id = None
                    time_limit = settings.ADHOC_QUERY_TIME_LIMIT

                result = execute_query.apply_async(args=(query, data_source.id, metadata, user_id, scheduled_query_id),
                                                   queue=queue_name,
                                                   time_limit=time_limit)
                job = QueryTask(async_result=result)
                tracker = QueryTaskTracker.create(
                    result.id, 'created', query_hash, data_source.id,
                    scheduled_query is not None, metadata)
                tracker.save(connection=pipe)

                logging.info("[%s] Created new job: %s", query_hash, job.id)
                pipe.set(_job_lock_id(query_hash, data_source.id), job.id, settings.JOB_EXPIRY_TIME)
                pipe.execute()
            break

        except redis.WatchError:
            continue
```
以上是Query功能的分析。主要组件：

- celery分布式框架
- redis缓存和任务队列
- postgre后台DB。

相关功能的设计比较中规中矩，报错异常的检测较少，后续值得优化。

## Model层

model层是Redash的设计核心，每个类对应后台数据库一张表和一个功能。

- access_permission, 权限表
- alembic_version,版本号	 	
- alert_subscriptions，报警描述
- alerts，报警列表	 	
- api_keys, api key管理
- changes，升级改动
- dashboards，报表存储
- data_source_groups,每个组对应的数据源
- data_sources，所有数据源
- events，后台日志
- groups，所有分组列表
- notification_destinations，报警的模板和目的地
- organizations， 组织	 	
- queries，所有queris	 	
- query_results，query的结果，另一种缓存
- query_snippets，SQL的评论	 	
- users, 用户已列表
- visualizations，可视化图表存储
- widgets，可视化控件

