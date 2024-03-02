# 深拷贝和浅拷贝

## 数据存储地址

javascript存在以下两种数据类型

* 基础数据类型「number、string、boolean、null、undefined、bigint、Symbol」
* 引用数据类型「object、array、function」

基础数据类型存储在栈区，存储的数据就是基础数据类型的值

引用数据类型数据存在堆区，栈区只存储了指向引用数据类型的地址

## 浅拷贝

只拷贝一层，更深层次的拷贝仍然指向同一片堆区地址，修改数据会同时修改拷贝前和拷贝后的数据

```javascript
const shallowClone = (value) => {
  // 只能拷贝对象类型，且不能是null
  if (typeof value !== "object" || value === null) {
    return value;
  }
 const data = new value.constructor();
  Reflect.ownKeys(value).forEach((key) => {
    data[key] = value[key];
  });
  return data;
};
```

类似浅拷贝的例子有

* Object.assign()
* 扩展运算符
* Array.prototype.slice()
* Array.prototype.concat()

## 深拷贝

完全拷贝数据，改变引用数据的值不会改变拷贝前后的数据

```javascript
const deepClone = (value, weakMap = new WeakMap()) => {
  if (typeof value !== "object" || value === null) {
    return value;
  }
  // 已拷贝的数据不用重复拷贝
  const obj = weakMap.get(value);
  if (obj) {
    return obj;
  }
  const data = new value.constructor();
  weakMap.set(value, data);
  Reflect.ownKeys(value).forEach((key) => {
    data[key] = deepClone(value[key], weakMap);
  });
  return data;
};
const obj = [
  {
    foo: 1,
    bar: {
      a: 2,
    },
  },
];
const cloneObj = deepClone(obj);
obj[0].bar.a = 3;
console.log(obj);
console.log(cloneObj);
```

除此之外，可使用下述方式完成深拷贝

* \_.cloneDeep()
* JSON.parse(JSON.stringify())

JSON.stringify()方式会导致undefined、function、symbol数据丢失

