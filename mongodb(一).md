---
title: "mongodb（一）"
date: 2021-01-27:07:07+08:00
tags: ["mongo", "database"]
categories: ["database"]
draft: false
---

#  mongodb 简易教程



>MogoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。
>
>在高负载的情况下，添加更多的节点，可以保证服务器性能。
>
>MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。
>
>MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。



### 基本概念与SQL区别

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| :----------- | :--------------- | :---------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接,MongoDB不支持                |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |



### 运行服务

`/bin`目录下的mongod.exe是开启服务的程序，第一次运行需要配置数据库目录

`$ mongod --dbpath xxxx`



### 运行shell

`/bin`目录下的mongo.exe是自带的shell，可以执行JavaScript代码

![](https://ftp.bmp.ovh/imgs/2021/01/08b7ac133091ac8d.png)

mongo会**隐式**创建数据库、集合



### 常用命令

* `show databases`   查看所有数据库
* `show  tables`\ show collections   查看所有集合
* `use dbname`   使用某个数据库
* `db`  查看当前数据库
* `db.createCollection('c1')`  创建c1集合
* `db.t1.drop()`  删除t1集合
* `db.dropDatabase()`  删除当前数据库
* `db.c1.find()`  查询当前数据库下c1集合
* `db.c1.insert({username:"zhangsan", age:19})`  插入一条数据
* `db.c1.insert([{username:"zhangsan", age:19}, username:"lisi", age:19}])`  插入多条数据



### 查询命令

| 操作       | 格式                     | 范例                                        |    RDBMS中的类似语句    |
| :--------- | :----------------------- | :------------------------------------------ | :---------------------: |
| 等于       | `{<key>:<value>`}        | `db.col.find({"by":"菜鸟教程"}).pretty()`   | `where by = '菜鸟教程'` |
| 小于       | `{<key>:{$lt:<value>}}`  | `db.col.find({"likes":{$lt:50}}).pretty()`  |   `where likes < 50`    |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `db.col.find({"likes":{$lte:50}}).pretty()` |   `where likes <= 50`   |
| 大于       | `{<key>:{$gt:<value>}}`  | `db.col.find({"likes":{$gt:50}}).pretty()`  |   `where likes > 50`    |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `db.col.find({"likes":{$gte:50}}).pretty()` |   `where likes >= 50`   |
| 不等于     | `{<key>:{$ne:<value>}}`  | `db.col.find({"likes":{$ne:50}}).pretty()`  |   `where likes != 50`   |
| 在...之间  | `{<key>:{$in:[value]}}`  | `db.c2.find({age:{$in:[5,8,10]})`           |     `where age in`      |

* `db.c2.find({}) / db.c2.find()`  查询全部数据
* `db.c2.find({},{username:1})`  查询全部数据  只显示`username` 字段
* `db.c2.find({},{username:0})`  查询全部数据  除了`username` 字段 其他全显示
* `db.c2.find({age:{$in:[5,8,10]})`  查询年纪在5，8，10岁之间的用户



### 修改命令

> 基本语法: `db.c1.update(<匹配条件>,<修改内容>)`

上述update其实是**替换**，会把没修改的字段清空，需要增加**修改器**

|  运算符   |        作用        |
| :-------: | :----------------: |
|  `$inc`   |        递增        |
| `$rename` | 重命名列（改字段） |
|  `$set`   |     修改列的值     |
| `$unset`  | 删除列（删除字段） |

* `db.c2.update({uname:"user0"},{$set:{uname:"user00"}})`  将`user0`名字改为`user00`
* `db.c2.update({uname:"user3"},{$inc:{age:4}})`  将user3年龄增加4岁
* `db.c2.update({uname:"user3"},{$unset:{age:true}})`  将user3年龄字段删除



**两点提示**

> 用的少: `db.c1.update(<匹配条件>,<修改内容>,[flag])`  flag为true **找不到则新增**

>默认只修改一条 即使匹配到多条: `db.c1.update(<匹配条件>,<修改内容>,[flag],[multi])`  multi为true**则修改多条**



### 删除命令

>基本语法：`db.c1.remove(<匹配条件>,[是否只删除一条 默认是删除多条])`

* `db.c3.remove({},true)`  只删除一条
* `db.c3.remove({})`  全部删除 



### 排序命令

`db.c2.find().sort({age:-1})`  根据age降序排列  -1为降序



### 分页命令

`db.c2.find().skip(数量).limit(数量)`  第一页时候可不写skip 



### 聚合查询

```
db.c1.aggregate([

​	{管道: {表达式}}

​	......

])
```

**常用管道**

|  管道名  |        作用        |
| :------: | :----------------: |
| `$group` |     将文档分组     |
| `$match` |      过滤数据      |
| `$sort`  | 聚合数据进一步排序 |
| `$skip`  |   跳过指定文档数   |
| `$limit` |   限制返回文档数   |



**常用表达式**

| 表达式 |  作用  |
| :----: | :----: |
| `$sum` |  求和  |
| `$avg` | 求平均 |
| `$min` | 求最小 |
| `$max` | 求最大 |



**根据性别分组，再对年龄求和**

```
db.c1.aggregate([
	{$group:{
		_id:"$sex",
		rs:{$sum:"$age"}
	}}
])
```



**统计男女生总人数**

```
db.c1.aggregate([
	{$group:{
		_id:"$sex",
		rs:{$sum:1} // 相当于count
	}}
])
```



**求学生总数和平均年龄**

```
db.c1.aggregate([
	{$group:{
		_id:null,
		total:{$sum:1},
		avg:{$avg:"$age"}
	}}
])
```



**查询男生、女生人数，按人数升序**

```
db.c1.aggregate([
	{$group:{
		_id:"$sex",
		rs:{$sum:1}
	}},
	{$sort:{rs:-1}}
])
```



### 索引

>索引是一种排序好的，便于快速查询的数据结构

* 创建索引语法：`db.集合名.createIndex(带创建索引的列,[额外选项])`

* 创建组合索引：`db.集合名.createIndex({键1:升降序,键2:升降序})`

* 创建唯一索引：db.集合名.createIndex({键1:升降序},{unique:"键1"})

* 删除索引

  >全部删除：`db.集合名.dropIndexes()`
  >
  >删除指定：`db.集合名.dropIndex(索引名)`

* 查看索引

  >`db.集合名.getIndexes()`



**练习**

* 给name添加普通索引  ` db.c1.createIndex({name:1})`  升序索引
* 删除name（升序）索引  `db.c1.dropIndex("name_1")`  
* 给name，age建立联合索引 `db.c1.createIndex({name:1,age:1})`
* 给name添加唯一索引  `db.c1.createIndex({name:1},{unique:"name"})`



### sql分析

>基本语法：查询语句.explain()
>
>COLLSCAN 全表扫描
>
>IXSCAN  索引扫描
>
>FETCH  根据索引去检索指定document

```
> db.c1.find({name:/a1$/}).explain(‘executionStats’)
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.c1",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "name" : {
                                "$regex" : "a1$"
                        }
                },
                "queryHash" : "420CA52A",
                "planCacheKey" : "420CA52A",
                "winningPlan" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "name" : {
                                        "$regex" : "a1$"
                                }
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]
        },
        ,
        "executionStats" : {
                "executionSuccess" : true, // 是否执行成功
                "nReturned" : 0,  //结果集数目
                "executionTimeMillis" : 0,  // 执行时间 毫秒
                "totalKeysExamined" : 0,// 索引检查时间
                "totalDocsExamined" : 3,  // 检查文档数
                "executionStages" : {
                        "stage" : "COLLSCAN", // 扫描方式
                        "filter" : {  // 过滤条件
                                "name" : {
                                        "$regex" : "a1$"
                                }
                        },
                        "nReturned" : 0,  // 返回结果集数
                        "executionTimeMillisEstimate" : 0,  // 预估时间
                        "works" : 5,  // 工作单元数
                        "advanced" : 0,  // 有心啊返回的结果集数
                        "needTime" : 4,
                        "needYield" : 0,
                        "saveState" : 0,
                        "restoreState" : 0,
                        "isEOF" : 1,
                        "direction" : "forward",
                        "docsExamined" : 3
                }
        },
        "serverInfo" : {
                "host" : "DESKTOP-DKSH7AL",
                "port" : 27017,
                "version" : "4.4.3",
                "gitVersion" : "913d6b62acfbb344dde1b116f4161360acd8fd13"
        },
        "ok" : 1
}
        "serverInfo" : {
                "host" : "DESKTOP-DKSH7AL",
                "port" : 27017,
                "version" : "4.4.3",
                "gitVersion" : "913d6b62acfbb344dde1b116f4161360acd8fd13"
        },
        "ok" : 1
}
```



### 权限机制

>必须切换到`admin`数据库`use admin`

**添加超级管理员**

```
db.createUser({
	"user":"admin",
	"pwd":"123456",
	"roles": [{
		role:"root",
		db:"admin"
	}]
})
```



**卸载服务**

>`/bin/mongod --remove`
>
>**DOS窗口必须用管理员身份运行**



**安装需要身份验证的mongodb服务**

>`/bin.mongod --install --dbpath D:\data\mangodb --logpath D:\Develoment\mangodb\log\mongodb2.log --auth`

>`net start mongodb`



**启动后通过超级管理员身份登录**

* 方法一：mongo 服务器ip:端口/**数据库** -u 用户名 -p 密码
* 方法二：先登录、**再选择数据库**、输入db.auth(用户名，密码)



**删除用户**

>`db.dropUser("football");`



**修改密码**

>`db.updateUser( "admin",{pwd:"password"});`



**角色**

| 角色           | 标识单词                                                     |
| -------------- | :----------------------------------------------------------- |
| 数据库用户角色 | read、readWrite                                              |
| 数据库管理角色 | dbAdmin、dbOwner、userAdmin                                  |
| 集群管理角色   | clusterAdmin、clusterManager、clusterMonitor、hostManager    |
| 备份恢复角色   | backup、restore                                              |
| 所有数据库角色 | readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase |
| 超级用户角色   | root                                                         |
| 内部角色       | __system                                                     |

* read：允许用户读取指定数据库
*  readWrite：允许用户读写指定数据库
* dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
* userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
*  clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
*  readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
* readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
*  userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的
* userAdmin权限
* dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。



**练习**

* 添加shop1用户 只能读shop

  ```
  db.createUser({
  	"user":"shop1",
  	"pwd":"123456",
  	"roles": [{
  		role:"read",
  		db:"shop"
  	}]
  })
  ```

  

* 添加shop2用户 只能读写shop

```
db.createUser({
	"user":"shop2",
	"pwd":"123456",
	"roles": [{
		role:"readWrite",
		db:"shop"
	}]
})
```

**！！！用户都是属于数据库的 所以登录、创建用户都要指定数据库**



### 备份和还原

需另外下载[命令工具包](https://www.mongodb.com/try/download/database-tools)

**备份**

>语法：mongodump -h -port -u -p -d -o



* 备份全部数据库

  >`mongodump -uadmin -p123456 -o D:\back`   需要root用户

* 备份指定数据库

  >`mongodump -dshop -ushop2 -p123456 -o D:\back`     只能用数据库的管理员 如果有的话



**还原**

>语法：`mongorestore -h -port  -u  -p  -d`
>
> `--drop` 先珊瑚数据库再导入



* 还原所有数据

  >`mongorestore -uadmin -p123456 --drop  D:\back`

* 还原指定数据库

  >`mongorestore -dshop -ushop2 -p123456 --drop  D:\back`\shop  指定数据库要有写权限的管理员



### 可视化工具

**studio 3t**  企业版 付费

**robo 3t**  免费