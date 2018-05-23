# Kylin应用篇


## 简介
Kylin是一款处理海量数据，提供SQL和多维度分析的OLAP工具。Kylin用于处理hadoop/spark场景下大量数据的预聚合，用户可自定义数据模型用于解决超过100亿+条记录的查询。


## 常见问题

### Not Support
- 建表或构建模型时，请勿使用中文列名。

- 不支持临时创建新列的统计

原表有两个字段a和b，通过concat进行拼接，然后做count()或者count(distinct *)。Kylin并不支持上述做法，因为无法命中相关的cube。

不能统计下述例子：

count(distinct concat(cast(a as varchar),b))

```
select count(distinct concat(cast(a as varchar),b))
from table_a
where dt = '2018-05-01'
```


### Cube命中

所有的SQL需要命中相关Cube才可以使用。如果不是使用姿势问题，请联系管理员构建新的Cube。

- 在join时，需要用事实表join维度表，负责容易出现：

```
No realization found for OLAPContext, CUBE_NOT_READY, CUBE_NOT_READY, CUBE_NOT_READY, MODEL_UNMATCHED_JOIN, MODEL_UNMATCHED_JOIN
```

- 在写子查询时，不能将事实表写在查询中，Cube可能无法命中。

## 调度脚本

```python
import datetime, requests

auth_str = "Basic YWRtaW46S1lMSU6="
url_str = "http://xxxxx.com:7070/kylin/api"


def auth():
    """
    用户认证
    :return:
    """
    url = "{url_str}/user/authentication".format(url_str=url_str)
    payload = "="
    headers = {
        'Content-Type': "application/x-www-form-urlencoded",
        'Authorization': auth_str,
        'Cache-Control': "no-cache"
        }
    response = requests.request("POST", url, data=payload, headers=headers)
    print(response.text)


def get_cube():
    """
    获取cube信息
    :return:
    """
    url = "{url_str}/cubes".format(url_str=url_str)
    querystring = {"cubeName": "test_join"}
    headers = {
        'Cache-Control': "no-cache",
        'Authorization': auth_str
        }
    response = requests.request("GET", url, headers=headers, params=querystring)
    print(response.json())


def build_cube(cube_name, start_date, end_date, build_type):
    """
    构建指定cube

    startTime 和 endTime 应该是utc时间。
    buildType 可以是 BUILD 、 MERGE 或 REFRESH。
        - BUILD 用于构建一个新的segment，
        - REFRESH 用于刷新一个已有的segment，
        - MERGE 用于合并多个已有的segment生成一个较大的segment

    :return:
    """
    url = "{url_str}/cubes/{cube_name}/rebuild".format(cube_name=cube_name, url_str=url_str)
    start_stamp = int(datetime.datetime.strptime(start_date, '%Y-%m-%d %H:%M:%S').timestamp() * 1000)
    end_stamp = int(datetime.datetime.strptime(end_date, '%Y-%m-%d %H:%M:%S').timestamp() * 1000)
    payload = "{\"startTime\": %d, \"endTime\": %d, \"buildType\": \"%s\"}" % (start_stamp, end_stamp, build_type)
    headers = {
        'Content-Type': "application/json",
        'Authorization': auth_str,
        'Cache-Control': "no-cache"
        }
    response = requests.request("PUT", url, data=payload, headers=headers)
    print(response.json())


if __name__ == '__main__':
    cube_name = "test_api"
    start_date = '2018-05-01 04:00:00'
    end_date = '2018-05-01 08:00:00'
    build_type = 'BUILD'
    build_cube(cube_name, start_date, end_date, build_type)
```
