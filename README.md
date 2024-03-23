---
description: graphql 是一种用于api 的查询语言
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 简介

## API

当我们请求一个用户的详情信息时，编写 gql 查询语句，按需获取用户数据

查询语句

```graphql
query User {
  user {
    name
    age
  }
}
```

响应数据

```json
{
  "user": {
    "name": "wei",
    "age": 17
  }
}
```

由此可以看出响应数据和查询语句有着相同的数据结构，通过查询语句就可以预测详情数据结构，与 restful api 完全不同的是，restful api 是资源定位数据，前端无法控制响应数据，而 graphql 提供了一种按需获取数据的解决方案

## 特点

* 按需无冗余的获取接口数据
* 多个资源，一次查询即可获取所有数据，无需发送多次接口拼接数据
* 基于类型系统，提供清晰错误提示和代码提示
