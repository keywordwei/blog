# 闭包

## 闭包是什么

闭包是由当前函数和它周围的词法环境组成的，函数创建时就会创建闭包，函数内部可以外部作用域中的变量。闭包具有以下特性：

* 函数内部可以外部作用域中的变量，存在作用域链引用关系
* 函数内部变量不受外部作用域干扰，可以模拟私有属性，私有方法
* 由于存在作用域引用关系，存在内存泄漏问题

示例

```javascript
const closure = () => {
  const a = 2;
  return () => {
    console.log(a);
  };
};
closure()(); // 2
```

## 循环中使用闭包

[门口](../)

## 闭包模拟私有方法

## References

[mdn 闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)
