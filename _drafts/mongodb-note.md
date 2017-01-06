---
layout: post
title: MongoDB笔记
date: 2017-01-06 11:15:44 +08:00
author: "izumo"
tags: 
    - MongoDB
---

# MongoDB安装、启动、关闭

## 安装MongoDB

从[官网](https://www.mongodb.com/)上下载编译好的二进制文件，解压至某个路径；

在系统路径PATH下添加`<mongo_install_location>/bin`；

创建数据文件目录`<mongo_data_location>/data/db`（随意）；

创建日志文件`<mongo_data_location>/data/mongodb.log`（随意）；

## 启动MongoDB

Linux下，启动MongoDB：

    mongod --dbpath=<mongo_data_location>/data/db --logpath=<mongo_data_location>/data/mongodb.log --logappend & 

> `&`表示后台运行（Really?）

## 配置MongoDB

每次启动MongoDB，都要输入上面那一堆代码，不免有些麻烦，MongoDB还可以通过配置文件方式启动。

    mongod --config <mongodb_config_file>

配置文件使用以下格式：

`<setting>=<value>`

部分设置：

#### logpath

日志文件位置

#### logappend

默认每次启动MongoDB都将新建一个日志文件，并覆盖旧文件。设置为`true`，则在原文件末尾附加。

#### port

监听客户端的端口，默认27017。

#### bind_ip

指定逗号分隔的ip地址列表，MongoDB在这些地址上侦听，默认为所有端口。

#### maxConns

最大连接数

#### auth

启用身份验证，默认为`false`。

#### noauth

禁用身份验证

#### rest

启用简单REST接口，能够通过REST请求来访问数据库，默认为`false`。

## 进入MongoDB Shell

    mongo

> MongoDB Shell使用JavaScript语法。

## 停止MongoDB

进入MongoDB Shell后，输入`db.shutdownServer()`。

# MongoDB文档格式

MongoDB使用类似JSON的BSON格式来储存数据：

    {
        <key_1>: <value_1>,
        <key_2>: <value_2>,
        <key_3>: [<value_3_1>, <key_3_2>, <key_3_3>],
        <key_4>: 
        {
            <key_4_1>: <value_4_1>,
            <key_4_2>: <value_4_2>
        }
    }

> 向MongoDB中插入文档时，会加入`_id`属性。

# MongoDB Shell操作

## 创建、切换数据库

    use <database_name>

*如果数据库不存在，则创建数据库，否则切换到指定数据库*

## 查看数据库

查看当前数据库：

    db

查看所有数据库：

    show dbs

## 删除数据库

切换到指定数据库，输入：

    db.dropDatabase()

## 创建集合

在数据库中插入文件时，指定集合名字，若集合不存在，则集合被自动创建。

    db.<collection_name>.insert(<document>)

或者使用以下方法：

    db.createCollection("<collection_name>")

## 删除集合

    db.<collection>.drop()

## 插入文档

    db.<collection_name>.insert(<document>)

    db.<collection_name>.save(<document>)

*使用`save()`方法时，若指定`_id`，则会覆盖原文档，否则和`insert()`方法相同。*

## 查找（所有）文档

    db.<collection_name>.find(<query>)

*`<query>`为空时，查找所有文档。*

## 更新文档

#### update方法

    db.<collection_name>.update(
        <query>,
        <update>,
        {
            upset: <boolean>,
            multi: <boolean>,
            writeConcern: <document>
        }
    )

参数：

+ query：查询条件，类似sql update查询where后面的。

+ update：update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的。

+ upsert：可选，如果不存在update的记录，是否插入objNew，true为插入，默认是false，不插入。

+ multi：可选，mongodb 默认是false，只更新找到的第一条记录，如果这个参数为true，就把按条件查出来多条记录全部更新。

+ writeConcern：可选，抛出异常的级别。

例：

    db.test.insert(
        {
            title: "MongoDB笔记",
            date: "2017-01-06 11:15:44 +08:00",
            author: "izumo"
        }
    )

使用`update()`方法来更新`title`：

    db.test.update(
        {'title': 'MongoDB笔记'},
        {$set: {'title': 'mongodb-note'}}
    )

#### save方法

    db.<collection_name>.save(
        <document>,
        {
            writeConcern: <document>
        }
    )

*`<document>`必须包含`_id`属性。*

## 删除文档

    db.<collection_name>.remove(
        <query>,
        {
            justOne: <boolean>
            writeConcern: <document>
        }
    )

*`justOne`参数表示是否只删除一个文档。*

使用以下方法删除集合中所有文档：

    db.<collection_name>.remove({})

# MongoDB Java Driver
