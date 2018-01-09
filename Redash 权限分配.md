# Redash权限分配



## 前言

Reash是一个数据查询平台，必定会涉及权限管理。主要由3个概念：组（group）,用户（user）， 数据源（data source）。

group在最上层，一个group对应多个user 和 data source。反之也可行，但不利于权限的管理。

权限功能主要基于组（group）和所属数据源(data source)来控制。一个用户（user）必须属于一个或多个组，当新用户进入时，该用户默认在“default组”。在新增一个数据源时，该数据源默认归属于"default组"。


## 数据源

数据源有两个权限设置：

-  Full access，即：该组的用户可操作被保存的查询（可以改SQL源码），创建一个新的查询
-  View Only access，即：该组的用户只能阅读被保存的查询（看不到SQL源码）及其结果

对于不需要写SQL取结果的用户群来说，应该与需要写SQL取结果的用户群体区分在不同组（group）中。比如：基础架构查看报表组，基础架构制作报表组。

如果想对一个组的用户做table级别的查询限制，官方提供的方案：

>The idea is to leverage your database’s security model and hence create a user with access to the tables/columns you want to give access to. Create a data source that's using this user and then associate it with a group of users who need this level of access.
>

翻译过来：在建立数据源时，配置一个带有权限控制的数据库用户即可。比如，连接mysql时，用一个只能查询测试db/table的用户名进行连接。把这个数据源赋给某个组，然后该组的所有用户，只能看到这个数据源里的测试db/table。

## 组

组可设置的权限有：

- admin/super_admin,管理员/超级管理员，可用所用功能
- create_dashboard,创建dashboard
- create_query,创建SQL查询
- edit_dashboard,编辑自己/别人的dashboard
- edit_query,编辑自己/别人的SQL查询
- view_query,查看已经存在的SQL
- view_source,查看SQL源码
- execute_query,执行SQL
- list_users,看到所有用户
- schedule_query,设置定时刷新
- list_dashboards,看到所用的dashboard
- list_alerts,看到所有的提醒任务
- list\_data_sources,看到所有的数据源

将以上权限赋给不同的组，每个组的用户就可以实现不同的功能。Redash不强调对用户做太多的权限控制，因为一个用户必须要归属于一个组。所以，对组做现在即可。
