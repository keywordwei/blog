# 裁切与合并

## 裁切

### 场景

一个表格展示的列过多时，表格列信息无法完全展示，用户可能只关注部分列的信息，表格提供自定义列功能，支持控制表格列的显隐。

<figure><img src=".gitbook/assets/Snipaste_2024-03-23_14-14-52.png" alt=""><figcaption></figcaption></figure>

更改表格列的显隐后，需要重新刷新表格数据。如果是 restful 接口，接口字段不会改变每次加载数据都会加载所有表格列的数据；而使用 graphql 接口的话，其设计原则之一就是无冗余的按需加载所需数据，更改表格列显隐后，只需要加载显示的列的数据，所以需要提供裁切功能当表格列变化时提供每次查询的 query 语句。

查询语句

```graphql
query user($login: String!, $last: Int!) {
  user(login: $login) {
    repositories(last: $last) {
      totalCount
      nodes {
        id
        name
        description
        createdAt
        url
        isFork
        languages(last: $last) {
          nodes {
            id
            color
            name
          }
        }
        defaultBranchRef {
          name
        }
      }
    }
  }
}
```

只显示仓库名后的查询语句

```graphql
query user($login: String!, $last: Int!) {
  user(login: $login) {
    repositories(last: $last) {
      totalCount
      nodes {
        id
        name
        url
      }
    }
  }
}
```

### 设计

裁切功能的目标是根据表格列按需裁切 query 语句。表格列如何和 query 语句产生关联呢？

query 语句中的每个字段都对应着一个确定的路径，如`name`的路径是`user.repositories.nodes.name`,列名都会指定`prop`标识数据来源，只需要让`prop`和路径建立映射关系，就可以通过路径按需裁切出 query 语句。

1.  列名和 query 语句中的字段路径建立映射关系\


    ```javascript
    {
      total: 'user.repositories.totalCount'
      id: 'user.repositories.nodes.id',
      name: 'user.repositories.nodes.name',
      url: 'user.repositories.nodes.url',
    };
    ```


2.  `id` 和 `total` 数据并不属于需要展示的列，但是确是必须存在的数据，提供 columns 配置项筛选出显示的表格列， columns 的内容应该是表格列的全集\


    ```javascript
    {
      total: 'user.repositories.totalCount'
      id: 'user.repositories.nodes.id',
      columns: {
        name: 'user.repositories.nodes.name',
        url: 'user.repositories.nodes.url',
      },
    };
    ```


3.  路径过于冗长，有没有简写的方式？由于 graphql 提供了 fragment 进行片段复用，可以使用 fragment 名称简写路径，缩短路径长度\
    \
    query 语句\


    ```graphql
    fragment repository on Repository {
      id
      name
      description
    }

    query user($login: String!, $last: Int!) {
      user(login: $login) {
        repositories(last: $last) {
          totalCount
          nodes {
            ...repository
          }
        }
      }
    }
    ```

    \
    简写路径\
    `user.repositories.nodes`路径可以直接使用 fragment 名称 `repository` 代替\


    <pre class="language-javascript"><code class="lang-javascript">{
    <strong>  total: 'user.repositories.totalCount'
    </strong>  id: 'repository.id',
      columns: {
        name: 'repository.name',
        url: 'repository.url',
      },
    };
    </code></pre>

{% hint style="warning" %}
多个同名 fragment 复用时，简写路径只会裁切第一次使用 fragment 的语句，其余同名 fragment 的字段需要使用完整路径
{% endhint %}

4.  拍平数据；路径除了作为裁切功能的依据，也可以用来获取响应中字段数据；约定字段数据是数组数据时解析为 list 属性下的对象数组\
    \
    响应数据\


    ```javascript
    {
      user: {
        totalCount: 10,
        repositories: {
          nodes: [
            {
              id: 'R_kgDOHTg3yg',
              name: 'vue-cli',
              url: 'https://github.com/keywordwei/vue-cli',
            },
            {
              id: 'R_kgDOHoJKyA',
              name: 'eslint',
              url: 'https://github.com/keywordwei/eslint',
            },
          ],
        },
      },
    }
    ```

    \
    根据列名路径映射关系拍平后的数据\


    ```javascript
    {
      total: 10,
      list: [
        {
          id: 'R_kgDOHTg3yg',
          name: 'vue-cli',
          url: 'https://github.com/keywordwei/vue-cli',
        },
        {
          id: 'R_kgDOHoJKyA',
          name: 'eslint',
          url: 'https://github.com/keywordwei/eslint',
        },
      ],
    };
    ```



### 实现

1. 根据列名按需筛选所需列名映射关系，打印出未配置的列名
2. 校验 query 语句的合法性，每次查询只能有一个 query 语句
3. 根据当前路径和 query 语句检查缓存中有没有裁切后的query语句，存在则直接从缓存返回结果
4. 展开 fragment 中的字段，将所有 fragment 中的字段合并至 query 语句中，删除 fragment 片段，在展开过程中记录 fragment 名称和其路径的映射
5. 根据 fragment 映射关系补全路径，提供完成的路径信息
6. 遍历 query  语句  AST 抽象语法，根据完整路径信息裁切 query  语句，返回裁切后的 query
7. 将结果存储至缓存中，缓存内容是源路径、query 语句和目标路径、目标 query 语句

拍平逻辑

递归遍历路径，获取路径数据，当路径数据时数组时，遍历数组获取路径数据

```javascript
import { get } from 'lodash'
/**
 * 将第一层数组数据解析为 list 字段下的对象数组
 */
const resolvePathValue = (data, fields) => {
  if (fields.length === 0) {
    return data
  }
  const value = get(data, fields[0])
  if (Array.isArray(value)) {
    const res = []
    for (let i = 0; i < value.length; i++) {
      res.push(resolvePathValue(value[i], fields.slice(1)))
    }
    return res
  } else {
    return resolvePathValue(value, fields.slice(1))
  }
}
export default (data, paths) => {
  const pathValue = {}
  for (const prop of Object.keys(paths)) {
    const fields = paths[prop].split('.') || []
    pathValue[prop] = resolvePathValue(data, fields, '')
  }
  const res = {}
  const list = []
  for (const prop of Object.keys(pathValue)) {
    const value = pathValue[prop]
    if (Array.isArray(value)) {
      // 拍平为 list 数组
      for (let i = 0; i < value.length; i++) {
        list[i] = list[i] || {}
        list[i][prop] = value[i]
      }
    } else {
      res[prop] = value
    }
  }
  if (list.length > 0) {
    res.list = list
  }
  return res
}
```

## 合并

### 场景

在微前端架构下，查询语句的某些字段是由不同微应用动态注入的，这些字段只属于微应用所以不能全部写在一个 query 语句里，这时就需要实现 query 语句的合并功能。

源 query 语句

```graphql
fragment repository on Repository {
  id
  name
  description
}

query user($login: String!, $last: Int!) {
  user(login: $login) {
    repositories(last: $last) {
      totalCount
      nodes {
        ...repository
      }
    }
  }
}
```

需要合并的 query 语句

```graphql
fragment repository on Repository {
 defaultBranchRef {
    name
  }
}

query user($login: String!, $last: Int!) {
  user(login: $login) {
    repositories(last: $last) {
      nodes {
        ...repository
      }
      totalDiskUsage
    }
  }
}
```

合并后的 query 语句

```graphql
fragment repository on Repository {
  id
  name
  description
  defaultBranchRef {
    name
  }
}

query user($login: String!, $last: Int!) {
  user(login: $login) {
    repositories(last: $last) {
      totalCount
      totalDiskUsage
      nodes {
        ...repository
      }
    }
  }
}
```

### 设计

将要注入源 query 语句的字段，按照源 query 语句的层级重写编写一个新的合并 query 语句，这样当 query 语句解析 为 AST 抽象语法树后，实际上就是在合并两棵树；更通俗的说就是按照层级递归合并两个对象，只不过合并逻辑是要符合 AST 语法树格式的。

```javascript
const source = {
  user: {
    repositories: {
      totalCount,
      nodes: {
        repository: {
          id: 'x',
          name: 'x',
        },
      },
    },
  },
};

const target = {
  user: {
    repositories: {
      totalDiskUsage,
      nodes: {
        repository: {
          defaultBranchRef: {
            name: 'xxx',
          },
        },
      },
    },
  },
};
```

### 实现

1. 合并 query 语句
2. 合并 fragment 片段
3. 集成合并后的 query 语句和 fragment 片段，返回合并后 query 语句
