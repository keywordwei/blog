# var、let、const 区别

## 变量提升

* var 定义的变量存在变量提升，编译时将变量声明提前至当前作用域最前面，允许先使用后声明变量
* let、const 不存在变量提升，必须先声明后使用变量，如果先使用后声明，将会出现暂时性死去，代码执行抛错

```javascript
console.log(foo); // undefined
var foo; // 变量声明提前，赋值操作不会提前，所以在这之前打印 foo 值为 undefineds
foo = 'foo';

console.log(bar); // Uncaught ReferenceError: Cannot access 'bar' before initialization"
let bar = 'bar';

console.log(a); // Uncaught ReferenceError:  Cannot access 'a' before initialization"
const a = 10;
```

## 块级作用域

let、const 会形成块级作用域，变量只在块级作用域内可访问。

```javascript
{
  var foo = 'foo';
  let bar = 'bar';
}
console.log(foo); // foo
console.log(bar); // ReferenceError: bar is not defined
```

for 循环中使用 var 定义索引下标时，代码执行后将打印 2 个 2，因为 var 定义的变量 i 是全局变量，for 循环结束后 i 的值为 2，函数执行时向上层作用域查找 i 的值就是 2。

```javascript
const funcs = [];
for (var i = 0; i < 2; i++) {
  funcs[i] = function () {
    console.log(i);
  };
}
for (const func of funcs) {
  func(); // 2, 2, 2
}
```

当把 var 改成 let 后，将会打印 0、1，这是因为 let 在 for 循环内部形成了 2  个块级作用域，每个作用域独立保存了 i 的值。

```javascript
const funcs = [];
for (let i = 0; i < 2; i++) {
  funcs[i] = function () {
    console.log(i);
  };
}
for (const func of funcs) {
  func(); // 0, 1
}
```

let 产生的块作用域如下所示：

```javascript
{
  i = 0;
  funcs[i] = function () {
    console.log(i);
  };
}

{
  i = 1;
  funcs[i] = function () {
    console.log(i);
  };
}
```

## 重复定义变量

* var 可以重复定义变量名
* let 、const 不能重复定义变量名，重复定义将会产生语法错误

```javascript
var foo;
var foo;

let bar；
let bar； // SyntaxError: Identifier 'bar' has already been declared
```

## const 常量值无法修改

let 定义的是变量，const 定义的是常量，也就是无法修改变量值。

```javascript
const foo = 'foo';
foo = 3; // TypeError: Assignment to constant variable.
```

当定义一个常量对象时，对象上的数据时可以修改的，因为常量值「对象的引用地址」存储在栈区，对象存储在堆区，不可修改的是栈区的值。

```javascript
const foo = {
  a: 'a',
};
foo.a = 3;
console.log(foo.a); // 3
```

如果同时也不允许修改对象数据，使用 `Object.freeze()` 方法冻结对象`({writable: false,configurable: false}),` 将不支持修改属性值，也不支持添加、删除属性。

```javascript
const obj = {
  prop: 42,
};

Object.freeze(obj);

obj.prop = 33; // 静默失败，严格模式抛错
console.log(obj.prop); // 42
```

:warning: `Object.freeze` 是浅冻结，只会冻结对象自身的属性，如果属性值是对象或者函数，属性值可以修改，如果需要冻结属性对像可以深冻结对象。

```javascript
const deepFreeze = (obj) => {
  for (const key of Reflect.ownKeys(obj)) {
    const value = obj[key];
    if (typeof value === 'object' || typeof value === 'function') {
      deepFreeze(value);
    }
  }
  return Object.freeze(obj);
};

const obj = {
  a: {
    b: 1,
  },
};

deepFreeze(obj);
obj.a.b = 2;

console.log(obj.a.b); // 1
```
