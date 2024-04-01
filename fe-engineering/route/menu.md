---
description: 当使用约定式路由配置后，路由自动生成，菜单页面通常会渲染路由所对应的组件，所以根据约定式路由规则，菜单也是可以自动生成的。
---

# sitemap

## 设计

文件名只包含路由 path 和组件地址信息，菜单最直观的需要菜单名称、icon 、权限等信息，很明显菜单的这些信息只通过文件名信息已经无法满足。所以需要在文件内部定义菜单信息，而 vue SFC 支持自定块， 通过定义 sitemap 块以 JSON 数据格式描述菜单信息。

```html
<sitemap lang="json">
{
  "title": "运行状态",
  "icon": "settings"
  "extension": {
    "permission": {
      "category": "monitor-status"
    }
  }
}
</sitemap>
```

sitemap 中文翻译过来就是站点地图，一个项目完整的 sitemap 文件包含了所有路由，但是并不是每个路由都会映射成菜单的，菜单还需要通过可视化拖拽工具将 sitemap 拖拽成菜单，支持拖拽形成菜单间的层级关系，如一级菜单、二级菜单，同时也支持改变菜单顺序。

sitemap 还可以生成面包屑，根据 sitemap 层级信息生成面包屑信息

例如存在 sitemap 如下

```json
[
  {
    "title": "用户管理",
    "key": "UserManagement",
    "path": "/user",
    "children": [
      {
        "title": "详情",
        "path": "/user/:id/detail"
      }
    ]
  }
]
```

面包屑信息显示为 用户管理/详情

生成后的 sitemap 信息可以根据路由 path 自动生成层级关系，但是如果需要修改层级关系很不方便，同时也没法直观查看 sitemap 信息，因此提供 vscode sitemap editor plugin，支持预览、生成、拖拽 sitemap。

## sitemap editor

sitemap editor 属于vscode [`CustomEditor`](https://code.visualstudio.com/api/extension-guides/custom-editors) 插件，识别 sitemap.json 文件后显示自定义的 editor

```json
{
  "activationEvents": [
    "onCustomEditor:sitemapEditor.edit",
    "workspaceContains:**/public/sitemap.json"
  ],
  "contributes": {
    "customEditors": [
      {
        "viewType": "sitemapEditor.edit",
        "displayName": "Sitemap Editor",
        "selector": [
          {
            "filenamePattern": "sitemap.json"
          }
        ],
        "priority": "option"
      }
    ]
  }
}
```
