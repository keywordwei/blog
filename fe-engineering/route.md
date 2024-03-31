# 约定式路由

## 路由配置

约定 src/pages 下的 .vue 文件自动生成路由，支持忽略目录和文件不参与路由生成逻辑。

```bash
└─ src
    └─ pages
        └─ user
            ├─ create.vue
            └─ index.vue
```

生成路由数据如下：

```javascript
const routes = [
  { path: '/user', component: User },
  { path: '/user/create', component: UserCreate },
]
```

## 动态路由

支持定义字符 "\__" 为前缀的文件名，_"\_id" 将解析为 ":id"，符合动态路由的数据格式。

```bash
└─ src
    └─ pages
        └─ user
            └─ _id
                ├─ detail.vue
                └─ edit.vue
```

生成路由数据如下：

```javascript
const routes = [
  { path: '/user/:id/edit.vue', component: UserEdit },
  { path: '/user/:id/detail.vue', component: UserDetail },
]
```

## 嵌套路由

当一个页面有多个 Tab 标签页，每个标签页都有自身

##
