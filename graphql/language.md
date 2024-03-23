# 查询和变更

## 字段(Fields)

查询数据需要定位到具体的字段，响应会返回与查询结构一致的字段数据

查询语句

```graphql
{
  user {
    name
  }
}
```

响应数据

```json
{
  "user": {
    "name": "wei"
  }
}
```

## 注释(Comments)

使用 # 符号实现代码注释功能

```graphql
# 代码注释
query User {
  user {
    name
  }
}
```

## 参数(Arguments)

查询通常是需要指定筛选条件的，每个字段可以指定查询参数筛选数据

```graphql
query User {
  # 查询年轻为17岁的用户名称
  user(age: 17) {
    name
  }
}

```

## 别名(Aliases)

响应数据字段冲突时，或者期望改变响应数据字段名称，使用别名可以帮助我们实现这个功能，而不用获取到响应数据后再对数据做重命名处理

```graphql
# 代码注释
query User {
  user(age: 17) {
    name: domain_name
  }
}
```

响应数据

```json
{
  "user": {
    "domain_name": "wei"
  }
}
```

## 片段(Fragments)

当一个查询语句过于庞大，或者某些字段被频繁的使用导致存在一定的代码重复，引入片段提供可复用单元，在需要使用片段的地方引入它，编写更清晰的代码

```graphql
# 定义 fragment
fragment apps on Application {
  name
  description
}

query User {
  user(age: 17) {
    name
    apps {
      # 引入 fragment
      ...apps
    }
  }
}
```

{% hint style="info" %}
片段不能自己引用自己，循环引用会造成查询失败
{% endhint %}

## 操作名称(Operation name)

如同函数一样可以通过函数名称直观的反应函数实现的功能，graphql 语句也鼓励提供操作名称语义化操作内容，操作名称前需要指定操作类型关键字，操作类型可以是 query、mutation、subscription

```graphql
# 操作类型 query
# 操作名称 User
query User {
  user {
    name
  }
}
```

## 变量(Variables)

通常参数都是动态改变的，如果使用指定参数的方式传递筛选条件，那么每次都需拼接生成新的 graphql 语句，而使用变量传递动态的参数能保证 graphql 语句有且只有一个；使用 $ 关键字声明变量，！代表参数是不可选的，参数可以设置默认值

<pre><code><strong>query User($age: Integer! = 17) {
</strong>  user(age: $age) {
    name
  }
}
</code></pre>

## 指令(Directives)

有时我们需要根据变量动态的改变查询结构，graphql 提供了两种内置指令支持这种场景

* @include(if: Boolean)：仅在参数为 true 时包含此字段
* @skip(if: Boolean)：仅在参数为 true 时跳过此字段

```
# $widthFriends: false
query User($age: Integer, $widthFriends: Boolean) {
  user(age: $age) {
    name
    friends @include(if: $widthFriends) {
      name
    }
  }
}
```

```json
{
  "name": "wei"
}
```

## 变更(Mutations)

支持查询数据，同样也需要支持变更数据，提供 mutataion 关键词变更数据，查询和变更不同的是字段查询是并行执行的，变更字段是线性执行的，和读写操作类似可以同时读数据但是不能同时写入数据

```
mutation CreateUser($user: InputUser!) {
  user(user: $user) {
    name
    age
  }
}
```

变更操作同样支持查询变更后的数据

```json
{
  "name": "wei",
  "age": 17
}
```

## References

[graphql 查询与变更](https://graphql.cn/learn/queries/)
