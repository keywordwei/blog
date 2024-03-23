---
description: >-
  常见的 graphql IDE 有 graphql 官方提供的 graphiql, 也有 altair 提供的 chrome
  商店插件、客户端应用；推荐使用开箱即用的 altair chrome 插件。
---

# IDE

## graphiql

[graphiql](https://github.com/graphql/graphiql/tree/main/packages/graphiql#readme) 是 graphql 官方提供的 IDE，支持以下功能：

* 实时预览 graphql api 文档
* 实时编辑 graphql 语句，提供代码高亮、代码补全、错误提示、格式化代码
* 设置查询参数，实时查看查询结果

[UI](https://graphql.github.io/swapi-graphql/) 简洁优雅，功能全面；但是需要二次开发部署，设置 graphql 服务地址，没法开箱即用的使用 graphiql IDE。

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## altair

altair 提供了 chrome 插件，客户端应用等多种使用方式，最方便的则是直接安装 [chrome 插件](https://chromewebstore.google.com/detail/altair-graphql-client/flnheeellpciglgpaodhkhmapeljopja)。altair 支持 graphiql 提供的大部分功能，altair 支持配置式设置 graphql 服务地址，是一款开箱即用的 graphql IDE。

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

### &#x20;使用示例

1.  设置 graphql 服务地址\
    \
    github graphql 服务地址为`https://api.github.com/graphql`\


    <figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
2.  设置 grqphql 服务请求头参数，一般请求头为认证信息\
    \
    github graphql 服务请求头如下如下所示\


    ```json
    {
      "Authorization": "TOKEN github-acesstoken"
    }
    ```

    \
    github accesstoken 需要在 github 内生成，操作如下图所示\


    <figure><img src=".gitbook/assets/Snipaste_2024-03-08_23-25-40 (4).png" alt=""><figcaption></figcaption></figure>

    \
    设置服务请求头\


    <figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
3.  根据 api 文档，编写 graphql 语句，查询数据\


    ```graphql
    # 查询语句
    query {
      viewer {
       location
      }
    }
    ```



    <figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

## References

* [github explorer](https://docs.github.com/en/graphql/guides/using-the-explorer)
