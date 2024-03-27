# 前端国际化

## 现状

在`vue`项目中使用了`vue-i18n` 实现项目国际化，使用方式如下所示：

1.  提供语言包初始化 i18n 实例，注册 i18n 插件\


    ```javascript
    // 提供语言包
    const messages = {
      'zh-CN': {
        message: {
          hello: '你好，{name}'
        }
      },
      'en-US': {
        message: {
          hello: 'hello,{name}'
        }
      },
    }

    // 创建 i18n 实例
    const i18n = VueI18n.createI18n({
      locale: 'zh-CN', // 当前语言
      fallbackLocale: 'zh-CN', // 兜底语言
      messages, 
    })

    const app = Vue.createApp({})
    app.use(i18n)
    app.mount('#app')
    ```
2.  注册完成后，会在组件实例上挂载`$t`、`$tc`、`$d`、`$n`等方法，所以就可以在模版内直接调用这些方法，将路径映射成对应语言文案\


    ```vue
    <p>{{ $t('message.hello', {name: 'world'}) }}</p>
    ```

    \
    渲染结果\


    ```vue
    <p>hello world</p>
    ```



在大型项目中使用该方案，存在诸多问题：

1. 代码不具有可读性；无法直观的从代码观察出路径替代的文案，无法通过文案直接查找到源码文件，需要开发者主动理解路径和语言包映射关系，效率偏低
2. 维护成本高，开发体验差；持续维护语言包，定义多语言文案路径，并在使用时再次编写文案路径，路径数量随着语言的增多而增多
3. 翻译人员翻译困难，翻译人员也需要理解路径映射关系翻译成目标语言文案

因此，设计一种新的国际化方案替换掉 vue-i18n 是很有必要的。

## 设计

### 生产语言包

提供命令行工具，根据源码生成语言包，命令行功能设计如下：

1.  标记文案\


    ```html
    <!--源码-->
    <span>详情</span>
    ```



    ```html
    <!--标记后-->
    <span >{{ $tx('详情') }} </span>
    ```
2.  抽取标记后的文案，生成 xliff 文件；xliff 文件存储了翻译信息，翻译人员通过xliff editor 打开文件完成翻译工作\


    <pre class="language-xml"><code class="lang-xml">&#x3C;trans-unit id="9b2fc35f82bacf628e117392030d1246">
    <strong>    &#x3C;source>详情&#x3C;/source>
    </strong>    &#x3C;target state="translated">Details&#x3C;/target>
        &#x3C;context-group purpose="location">
          &#x3C;context context-type="sourcefile">file/index.js&#x3C;/context>
          &#x3C;context context-type="linenumber">26&#x3C;/context>
        &#x3C;/context-group>
        &#x3C;context-group purpose="location">
          &#x3C;context context-type="sourcefile">file/index.vue&#x3C;/context>
          &#x3C;context context-type="linenumber">27&#x3C;/context>
        &#x3C;/context-group>
    <strong>&#x3C;/trans-unit>
    </strong></code></pre>
3.  根据 xliff 生成语言包\


    ```json
     // en-US.json
     {
        "9b2fc35f82bacf628e117392030d1246": "Details"
      }
    ```

### 消费语言包

1. 项目启动时根据语言环境按需加载语言包「包括组件库语言包」
2. 在 vue 实例挂载标记方法 `$tx`，支持在 `SFC` 文件渲染成目标语言文案
3. 在全局变量挂载 i18n.t 方法，支持在脚本文件渲染成目标语言文案
