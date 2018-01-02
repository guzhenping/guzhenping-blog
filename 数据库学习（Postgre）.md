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
