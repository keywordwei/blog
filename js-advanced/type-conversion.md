# 比较运算

## 相等比较

* 宽松比较（==）
* 严格比较（===）
* 等值比较（Object.is）

## 特殊的类型转换

转换为number

<table><thead><tr><th>原始类型</th><th width="228">目标类型</th><th>结果</th></tr></thead><tbody><tr><td>number</td><td>boolean</td><td>0和NaN转换为false,其余转换成true</td></tr></tbody></table>

转换为string

<table><thead><tr><th>原始类型</th><th width="228">目标类型</th><th>结果</th></tr></thead><tbody><tr><td>undefined</td><td>string</td><td>"undefined"</td></tr><tr><td>null</td><td>string</td><td>"null"</td></tr><tr><td>array</td><td>string</td><td>数组的中的每一项以逗号连接</td></tr><tr><td>objcet</td><td>string</td><td>"[object, Object]"</td></tr></tbody></table>

转化为数字

<table><thead><tr><th>原始类型</th><th width="228">目标类型</th><th>结果</th></tr></thead><tbody><tr><td>array</td><td>number</td><td>[]、[undefined]、[null]、['']转换为0，其他转换为NaN</td></tr></tbody></table>

