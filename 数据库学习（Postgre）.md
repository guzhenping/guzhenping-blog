# 数据库学习（Postgre）
-------
## 前言


## sql基础

- 修改字段类型：

```
-- 将string类型变成int

LTER TABLE the_table ALTER COLUMN col_name TYPE integer USING (col_name::integer);    -- 字段值没有空格

ALTER TABLE the_table ALTER COLUMN col_name TYPE integer USING (trim(col_name)::integer);    -- 字段值有空格

ALTER TABLE onetruereport.report_jinrong_submit 
ALTER COLUMN createdtime TYPE integer USING (trim(createdtime)::integer);
```

- 转多列

```

SELECT unnest(string_to_array('
  xxx@xx.com,
  ccc@cc.com', ',')) as email 
```

string_to_array：将字符串处理成array数组，unnest将array转成多列。