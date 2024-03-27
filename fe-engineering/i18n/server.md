# 接口数据国际化

## 场景

接口数据如一些枚举值，需要映射成语义化的文案；timestamp 时间语义化成可读的时间

源数据

```javascript
{
  enum: 1,
  timestamp: 1604990724,
  file_size: 1024
}
```

目标数据

```javascript
{
  enum: '是',
  timestamp: '2024-03-26',
  file_size: '1kb'
}
```

这些数据也是需要支持国际化显示到UI上，并且这些数据还要支持导出功能的话，导出的数据也要支持国际化；很明显 UI 数据的国际化是需要前端实现的，导出数据由于前端无法拿到全量的数据，导出数据的国际化是放在后端实现的；但是前端和后端最后呈现的文案是一致，也就是数据翻译逻辑一致的，就需要提供一种统一的方案，使前端和后端都根据一套翻译逻辑，实现接口数据国际化。

## 设计

前端实现翻译逻辑，后端使用无头浏览器在 JS 环境调用翻译方法，前后端统一使用一套翻译逻辑。

1.  支持模版配置式实现翻译逻辑，内置 4 种翻译数据类型，enum、timestamp、filesize、Date，也支持自定数据翻译逻辑\


    ```javascript
    const templates = [
      {
        category: 'category',
        fields: [
          {
            name: 'time',
            type: TIMESTAMP,
            units: TIMESTAMP_UNITS.S,
            formatter: 'YY-MM-DD HH:mm:ss',
          },
          {
            name: 'enum',
            type: ENUM,
            alias: 'locales path',
          },
          {
            name: 'date',
            type: DATE,
            formatter: 'YY-MM-DD HH:mm:ss',
          },
          {
            name: 'file_size',
            type: FILE_SIZE,
          },
          {
            name: 'custom translate logic',
            type: CUSTOM,
            formatter: ({ data, locales }) => {},
          },
        ],
      },
    ];
    ```


2.  通过依赖包维护枚举值的语言包，根据语言包自动生成枚举值模版，所以可以省略枚举值模版配置\


    ```javascript
    const locales = {
      'zh-CN': {
        [category]: {
          [field]: {
            0: '是',
            1: '否',
          },
        },
      },
      'en-US': {
        [category]: {
          [field]: {
            0: 'Yes',
            1: 'No',
          },
        },
      },
    };
    // 根据语言包自动生成的 templates
    const templates = [
      {
        category,
        fields: [
          {
            name: field,
            type: ENUM,
          },
        ],
      },
    ];
    ```


3.  提供 translate、translateCol、translateField 方法根据数据和模版实现数据翻译\


    ```javascript
    translate({
      locale: String,
      category: String,
      data: list,
    });

    translateCol({
      locale: String,
      category: String,
      data: Object,
    });

    translateField({
      locale: String,
      category: String,
      field: String,
      data: any,
    });
    ```
4. mock 数据检验翻译逻辑是否正确
