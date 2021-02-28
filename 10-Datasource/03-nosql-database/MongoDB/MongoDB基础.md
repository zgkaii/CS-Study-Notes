# 1. MongoDB简介

MongoDB 是一个高性能，开源，无模式的文档型数据库，是当前 NoSQL 数据库产品中最热门的一种。它在许多场景下可用于替代传统的关系型数据库或键/值存储方式，MongoDB 使用 C++开发。MongoDB 的官方网站地址是：http://www.mongodb.org/。

## 1.1 NoSQL



## 1.2 MongoDB存储模型

### 逻辑结构

MongoDB 的逻辑结构是一种层次结构。主要由：文档(document)、集合(collection)、数据库(database)这三部分组成。

<img src="..\..\..\images\MongoDB\逻辑结构.png" alt="img" style="zoom: 80%;" />

与关系型数据库对比：

| MongoDB          | 关系型数据库     |
| ---------------- | ---------------- |
| 数据库(database) | 数据库(database) |
| 集合(collection) | 表(table)        |
| 文档(document)   | 行(row)          |

MongoDB 存储的数据类型为 BSON，BSON 与 JSON 比较相似，文档存储模型也支持数组和键值对。

![img](..\..\..\images\MongoDB\文档.svg)

## 1.3 BSON VS JSON

# 2. MongoDB CRUD

MongoDB的安装与启动可查看[安装 MongoDB 社区版](https://www.docs4dev.com/docs/zh/mongodb/v3.6/reference/administration-install-community.html)。

## 2.1 数据库、集合与文档

## 2.2 插入数据

## 2.3 更新数据

## 2.4 删除数据

## 2.5 查询数据

# 3. 高级查询

# 4. 索引

# 5. Java操作MongoDB