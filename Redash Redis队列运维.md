# Redash 队列运维

```python
# coding:utf-8
import datetime, json, redis

r = redis.StrictRedis(host='xx.xxx.xx.xx', port=123, db=1)

list = ['query_task_trackers:waiting', 'query_task_trackers:in_progress']
pipe = r.pipeline()

for item in list:
    t = item
    ids = r.zrevrange(t, 0, -1)
    print("The queue " + item + " has " + str(len(ids)))
    if len(ids) == 0:
        continue
    for id in ids:
        pipe.get(id)
        p = pipe.execute()
		  
        # 可以调试看看p的数据结构，现阶段值会返回list，index=0
        try:
            json_text = json.loads(p[0])
        except Exception as e:
			  pass

        timestamp = json_text['created_at']
        date = datetime.datetime.fromtimestamp(timestamp)
        date_str = date.strftime("%Y-%m-%d %H:%M:%S")
        print(date_str)
        
        # 选择自己合适的时间
        flag_day = datetime.datetime.today() - datetime.timedelta(days=1)
        # flag_day = datetime.datetime.today() - datetime.timedelta(hours=12)
        print(flag_day)
        if flag_day > date:
            print("可以删除...")
            print("delete task: " + str(id))
            r.delete(id)
            r.zrem(t, id)
            print("deleted success...")
        else:
            print("不需要删除...")

```
