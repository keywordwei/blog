# 原型链

## 原型链是什么

每个对象都有`__proto__`属性，代表对象的原型，原型的值也就是`__proto__`属性值也是一个对象，所以原型对象也会有 `__proto__`属性，一直到`__proto__`原型对象的值为`null` 时，整个原型链才结束；这种一层层嵌套`__proto__`原型对象的方式就产生了原型链，原型链的终点是`null`。

```javascript
const foo = {
  a: 1,
  __proto__: {
    b: 2,
    __proto__: {
      c: 3,
      __proto__: null
    }
  }
}
```

**对象可以访问原型链对象上的属性**\
例如当我们访问对象属性时，例如查找`foo.a`的值，先会查找 foo 对象自身有没有属性 a，如果没有则会在对象的`__proto__`原型对象上查找属性 a，直到`__proto__`原型对象为 null 结束查找，查找过程过程存在属性 a 则返回它的值，如果没有返回 undefined。

```javascript
const foo = {
  a: 1,
  __proto__: {
    b: 2,
    __proto__: {
      c: 3,
      __proto__: null
    }
  }
}

console.log(foo.a); // 1
console.log(foo.b); // 2
console.log(foo.c); // 3
console.log(foo.d); // undefined
```

## 构造函数

除箭头函数外的函数都存在`prototype`属性，当函数被作为构造函数使用时，产生的实例对象的`__proto__`的值就是构造函数`prototype`值，也就是`instance.__proto__ === Constructor.prototype`

```javascript
const arrow = () => {};
const Foo = function () {};
// 箭头函数没有 prototype 属性
console.log(arrow.prototype); // undefined
console.log(Foo.prototype); // {}

const instance = new Foo(1);

console.log(instance instanceof Foo); // true
// 等价于
console.log(instance.__proto__ === Foo.prototype); // true
```

构造函数的`prototype.constructor`值就是构造函数本身，所以实例对象都能通过 `constructor` 属性访问对象的构造函数

<pre class="language-javascript"><code class="lang-javascript"><strong>function Foo(a) {
</strong>  this.a = a;
}
const instance = new Foo(1);

// 构造函数的 prototype.constructor
console.log(Foo.prototype.constructor === Foo); // true
console.log(instance.constructor === Foo); // true
// 等价于
console.log(instance.__proto__.constructor === Foo); // true
</code></pre>

构造函数的`prototype`值为一个对象，`prototype`的原型链是什么？

`prototype.__proto__`的值等价于构造`prototype`对象的构造函数的`prototype`值，`prototype`对象是通过`Object`函数构造的，所以原型链为 `Object.prototype -> null`

```
// Foo.prototype 的原型链
console.log(Foo.prototype);
console.log(Foo.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__ === null); //true
```

构造函数也是一个对象，构造函数的原型链是什么？

构造函数的`__proto__`等价于构造它的构造函数的`prototype`值，而函数的构造函数是Function，所以原型链为 `Function.prototype -> Object.prototype -> null`

```
// Foo.__proto__ 的原型链
console.log(Foo.__proto__);
console.log(Foo.__proto__ === Function.prototype); // true
console.log(Function.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__ === null); //true
```

## 产生原型链的几种方式

### 字面量式创建对象

```javascript
const foo = {};
const arr = [];
const func = function () {};

console.log(foo.__proto__ === Object.prototype); // true
console.log(arr.__proto__ === Array.prototype); // true 
console.log(func.__proto__ === Function.prototype); // true
```

### new

使用 new 构造实例对象时，实例对象的\_\_proto\_\_ === 构造函数的`prototype`,new 一个类也是一样的，类本身就是构造函数的语法糖

```
function Foo(a) {
  this.a = a;
}

const instance = new Foo(1);

console.log(instance instanceof Foo); // true
// 等价于
console.log(instance.__proto__ === Foo.prototype); // true

class Bar {}
const bar = new Bar()
console.log(bar.__proto__ === Bar.prototype); // true
```

### Object.create

`Object.create(prototype, properties)`函数可以生成一个新对象，新对象的`__proto__ === prototype`参数，也就是可以指定生成的对象原型是什么，我们可以使用它来实现继承

```javascript
// 父类
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function () {
  console.log('animal ', this.name);
};
// 子类
function Cat(name) {
  Animal.call(this, name);
}

Cat.prototype = Object.create(Animal.prototype, {
  constructor: {
    value: Cat,
    enumerable: false,
    writable: true,
    configurable: true,
  },
});
const rect = new Cat('cat');

console.log(rect.speak()); // cat
console.log(rect instanceof Cat); // true
console.log(rect instanceof Animal); // true
console.log(rect.constructor === Cat); // true
```

{% hint style="warning" %}
Object.create 会导致 Cat.prototype.constructor 丢失，所以需要指定 constructor 的值为 Cat
{% endhint %}

### Object.setPrototypeOf

与`Object.create`类似也是设置对象的原型，`Object.create`可以使用下面代码代替，效果也是一样的

```javascript
Object.setPrototypeOf(Cat.prototype, Animal.prototype);
```

### class extends

ES6 class 是构造函数的语法糖，使用 extends 方式实现类的继承

```javascript
// 父类
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    console.log('animal ', this.name);
  }
}
// 子类
class Cat extends Animal {
  constructor(name) {
    super(name);
  }
}

const cat = new Cat('cat');

console.log(cat.speak()); // cat
console.log(cat instanceof Cat); // true
console.log(cat instanceof Animal); // true
console.log(cat.constructor === Cat); // true
```

{% hint style="warning" %}
class 的代码时在严格模式执行的，当 class 静态方法或者实例方法以默认方式执行时，this 指向undefined
{% endhint %}
