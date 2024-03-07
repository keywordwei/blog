---
description: this绑定的值需要在函数调用的时候分析，this 值是在运行时进行绑定的，有默认绑定、隐私绑定、显示绑定、new 绑定、箭头函数绑定5种方式
---

# this 绑定

## 默认绑定

函数直接执行，没有任何前缀方式执行，this 在非严格模式下指向 globalThis (同一标准获取不同环境下的全局对象)，浏览器环境指向 window，Node 环境指向 global；this 在严格模式下指向 undefined

```javascript
var a = 'globalThis';
function foo() {
  console.log(this, this.a);
}
function bar() {
  'use strict';
  console.log(this);
}
foo(); // window,globalThis
bar(); // undefined
```

## 隐式绑定

通过访问对象成员方式调用函数，this 指向 "." 前最近的对象，隐式绑定可能会导致 this 绑定数据丢失

```javascript
const obj = {
  foo: 'foo',
  bar: {
    foo: 'bar',
    fn() {
      console.log(this.foo);
    },
  },
};

obj.bar.fn(); // bar
const fnCopy = obj.bar.fn;
fnCopy(); // undefined
```

## 显示绑定

由于隐式绑定存在 this 数据篡改的情况，有没有一种方式可以保证 this 值不被篡改呢，使用 Function.prototype.call、Function.prototype.apply、Function.prototype.bind 可以显示设置 this 绑定的值并且无法篡改

```javascript
function sum(a, b) {
  console.log(this.c + a + b);
}
const test = {
  c: 3,
};
sum.call(test, 1, 2); // 6
sum.apply(test, [1, 2]); // 6
const sumCopy = sum.bind(test);
sumCopy(1, 2); // 6
sumCopy.bind({ c: 4 })(1, 2); // 6
```

在非严格模式下，当绑定的值是非对象时，this 指向类型转化后的对象，null、undefined 指向 globalThis

```javascript
function sub() {
  console.log(Object.prototype.toString.call(this));
}
sub.call(1); // [object Number]
sub.apply('2'); // [object String]
sub.call(undefined); // [object global]
```

### call、apply、bind 区别

* call、apply 调用即执行函数，bind 返回函数的拷贝，返回一个全新的函数，call、apply 执行后需要再次进行 this 绑定，bind 只需一次就能完成 this 绑定
* call 以散列平铺的方式传递函数参数，apply 则以数组方式传递参数，性能方面 call 执行效率高于 apply ，因为 apply 需要处理将数组参数散列平铺逻辑

## new 绑定

new 方式实例化对象时，this 指向构造函数返回的对象

```javascript
class Book {
  constructor() {
    this.page = 23;
  }
  printPage() {
    console.log(this.page);
  }
}

const book = new Book();
book.printPage(); // 23
function Person() {
  this.age = 22;
  return {
    age: 17,
  };
}

const student = new Person();
console.log(student.age); // 17
```

实现 new 操作

## 箭头函数绑定

箭头函数 this 指向外层作用域绑定的 this, 调用时即绑定且无法更改

```javascript
var b = 'b';
const arrow = () => {
  console.log(this.b);
};
arrow(); // b
```

## Dom 事件处理器中的 this 绑定

dom 事件中 this 指向绑定事件的 dom 元素，这种绑定方式类似隐式绑定

```javascript
// 当作为监听器调用时，将相关元素变为蓝色
function bluify(e) {
  // 总是为 true
  // e.currentTarget 事件绑定的元素
  console.log(this === e.currentTarget);
  // 当 currentTarget 和 target 是同一个对象时为 true
  // e.target 事件触发的元素
  console.log(this === e.target);
  this.style.backgroundColor = '#A5D9F3';
}

// 获取文档中的每一个元素
const elements = document.getElementsByTagName('*');

// 添加 bluify 作为点击监听器，所以当元素被点击时，它会变蓝
for (const element of elements) {
  element.addEventListener('click', bluify, false);
}
```

内联事件绑定的是当前元素

```javascript
<button onclick="alert(this.tagName.toLowerCase());">Show this</button>
// button,this 指向 button 元素
<button onclick="alert((function(){return this})());">Show inner this</button>
// this 指向 window，执行的时候属于默认绑定方式
```

## 绑定优先级

new绑定  > 显示绑定 > 隐式绑定 > 默认绑定

```javascript
function Base(foo) {
  this.foo = foo;
}
const param = {};
const baseCopy = Base.bind(param);
baseCopy('foo');
console.log(param.foo); // foo
const baseObj = new baseCopy('bar');
console.log(param.foo); // foo
console.log(baseObj.foo); // bar
```

* Function.prototype.bind 函数实现时会判断函数是否被 new 调用，如果被 new 调用则 this 指向 new操作后的对象；
* 实现 bind 函数

## References

* <<你不知道的js>>
* [mdn this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)
* [js this 绑定](https://www.cnblogs.com/echolun/p/11962610.html)

