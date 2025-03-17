+++
title = "摆脱CRUD-实现数据库（一）"
date = 2025-03-14T11:54:00+08:00
tags = ["Database"]
categories = ["摆脱CRUD系列"]
draft = false
IMAGE = "/images/pl-banner.jpg"
AUTHOR = "LW"
+++

-   设计实现一款怎么样的数据库
    1.  实现几款基本的关系型数据库
    2.  能够执行基本的查询语句
    3.  能够支持基本的事务
-   如何快速实现
-   使用什么语言实现
    尽量不要将精力放在编程语言上，应该在数据库的技术实现和工程化方面，并且尽量简单和流行
    1.  python
    2.  rust/go (备选)
-   要点和难点有哪些
    1.  理解数据库的执行计划
    2.  对数据库事务实现机制的理解


## 项目目录结构 {#项目目录结构}

sql： sql 引擎

-   parser： 用于 sql 语句的解析
-   optimizier： 用于生成 sql 语句过程进行执行计划的优化

excutor： 执行引擎

-   operator： 执行算子，执行计划的最小单位

storage： 存储引擎，用于存储和组织数据

-   transaction： 用于事务管理

network: 网络通信层
common： 公共组件部分，工具等


## 相关API {#相关api}

-   数据库 `sql` 语句总收入口： `exec_deppsdb_query(sql_stmt: str) -> Result`
    用于接收用户的访问 `SQL`
-   SQL引擎SQL解析部分的入入口 `query_parse(sql_stmt:str)->ASTNode`
-   SQL引擎优化过程入口 `query_plan(ast:ASTNode)->PlanTree`
-   执行引擎的入口 `exec_paln(plan_tree:PlanTree)->Result`
