---
title: javascript new 原理
date: 2020-03-17 18:12:26
tags: javascript
categories: 技术水波文
---

## new 的作用

我们先通过简单的例子来了解下 `new` 的作用吧。

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.getName = function () {
    console.log(this.name);
}
var person = new Person('xyue');
console.log(person); // Person {name: "xyue"}
console.log(person.name); // 'xyue'
console.log(person.getName()); // 'xyue'
```

从上边例子中我们可以得到结论：

+ 通过 `new` 构造函数 `Person` 创建出来的实例可以访问到构造函数中的属性
+ 通过 `new` 构造函数 `Person` 创建出来的实例可以访问到构造函数原型链中的属性，也就是说通过 `new` 操作符，实例与构造函数通过原型链连接了起来

但是当前的构造函数 `Person` 并没有 `return` 任何值，如果我们让它返回值会发生什么事情呢？

```javascript
function Person(name) {
  this.name = name;
  return 1;
}
var person = new Person('xyue');
console.log(person); // Person {name: "xyue"}
console.log(person.name); // 'xyue'
```

虽然在构造函数中返回了 `1`，但是这个返回值（原始值）并没有任何用处，得到的结果完全一样。
所以通过这个例子，我们可以得出另外一个结论：

+ 构造函数如果返回原始值，那么这个返回值没有意义

试完了返回原始值，我们再来试试返回对象会发生什么事情吧：

```javascript
function Person(name) {
  this.name = name;
  console.log(this); // Person {name: "xyue"}
  return { age: 18 };
}
var person = new Person('xyue');
console.log(person); // {age: 18}
console.log(person.name); // undefined
```

通过这个例子我们可以发现，当构造函数返回值为对象时，内部的 `this` 虽然正常工作，但是这个返回值会被正常的返回出去。
通过这个例子，我们再次得到一个结论：

+ 构造函数如果返回值为对象，那么这个返回值会被正常使用

## 实现

通过以上几个例子，我们大致了解了 `new` 操作符的几个作用

> **`new` 运算符** 创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。`new` 关键字会进行如下的操作：

1. 创建一个空的简单 JavaScript 对象（即`{}`）
2. 链接该对象（即设置该对象的构造函数）到另一个对象
3. 将步骤 1 新创建的对象作为 `this` 的上下文
4. 如果该函数没有返回对象，则返回 `this`

```javascript
function _new() {
    // 创建的新对象
    let obj = {};
    // 第一个参数是构造函数
    let [constructor, ...args] = [...arguments];
    // 执行[[原型]]连接;obj 是 constructor 的实例
    obj.__proto__ = constructor.prototype;
    // 执行构造函数，将属性或方法添加到创建的空对象上
    let result = constructor.apply(obj, args);
    if (result && (typeof result === 'object' || typeof result === 'function')) {
        // 如果构造函数执行的结构返回的是一个对象，那么返回这个对象
        return result;
    }
    // 如果构造函数返回的不是一个对象，返回创建的新对象
    return obj;
}

_new(Person,'xyue'); // Person {name: "xyue"}
```

通过 `new` 操作符，我们可以创建原对象的一个实例对象，而这个实例对象继承了原对象的属性和方法，所以 `new` 存在的意义在于它实现了 `javascript` 中的继承，而不仅仅是实例化了一个对象。
