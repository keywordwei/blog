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

当一个页面有多个 Tab 标签页，每个标签页都需要渲染对应的组件，切换 tab 标签就需要使用嵌套路由，约定创建与目录名同名且同层级的 vue 文件时生成嵌套路由。\


<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

```bash
└─ src
    └─ pages
        ├─ user
        │   └─ _id
        │       ├─ detail.vue
        │       └─ edit.vue
        └─ user.vue
```

生成路由如下所示

<pre class="language-javascript"><code class="lang-javascript">const routes = [
  {
<strong>    path: '/user',
</strong>    component: User,
    children: [
      { path: '/user/:id/edit.vue', component: UserEdit },
      { path: '/user/:id/detail.vue', component: UserDetail }
    ]
  }
]
</code></pre>

## 嵌套结构代码实现

将匹配到的路由文件根据片段长度升序排列，递归遍历文件名查找属于当前文件路径前缀的文件，如果存在则是嵌套路由放入 children 数组；遍历过程中记录已经匹配到前缀的文件名跳过文件遍历流程。

```javascript
const path = require('path');
const fg = require('fast-glob');
const traverse = (files, set, prefix = '') => {
  const nestedFiles = [];
  for (const file of files) {
    if (set.has(file) || !file.startsWith(prefix)) {
      continue;
    }
    const options = {
      file,
    };
    set.add(file);
    const _prefix = file.replace(path.extname(file), '');
    const children = traverse(files.slice(1), set, _prefix);
    if (children.length > 0) {
      options.children = children;
    }
    nestedFiles.push(options);
  }
  return nestedFiles;
};

const main = async () => {
  const patterns = ['src/pages/**/*.vue'];
  const files = await fg(patterns);
  // 根据文件片段长度升序排序
  files.sort((a, b) => {
    return a.split('/').length - b.split('/').length;
  });
  const set = new Set();
  console.log(JSON.stringify(traverse(files, set), null, 2));
};
main();
```

