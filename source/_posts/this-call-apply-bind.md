---
title: this、call、apply、bind
date: 2020-03-18 15:16:37
tags: javascript
---

![](/this-call-apply-bind/header-img.png)

## this

面向对象语言中 `this` 表示当前对象的一个引用。
但在 `JavaScript` 中 `this` 不是固定不变的，它会随着执行环境的改变而改变。

### 普通函数指向函数的调用者

#### 默认绑定

```javascript
var a = 'xyue';
function foo() {
    console.log(this.a);
}
foo(); // xyue
```
`foo()` 直接调用，非严格模式下 `this` 指向 `window` ，严格模式下 `this` 指向 `undefined`;

#### 隐式绑定

```javascript
var a = 'xyue1';
var obj = {
    a: 'xyue2',
    foo() {
        console.log(this.a);
    }
}
obj.foo(); // xyue2
var bar = obj.foo; 
bar(); // xyue1
```

`obj.foo()` 是 `obj` 通过 `.` 运算符调用了 `foo()`，所以 `this` 指向  `obj`；
`bar()` 实际上是把 `foo` 函数赋值给了 `bar`，没有调用者，所以使用的是默认绑定规则，这里的 `this` 指向 `window`。

#### 显式绑定

```javascript
var a = 'xyue1';
var obj = {
    a: 'xyue2',
    foo() {
        console.log(this.a);
    }
}
var bar = obj.foo; 
bar(); // xyue1
bar.call(obj) // xyue2
```

使用 `call`、 `apply` 、 `bind` 可以显式修改 `this` 的指向。

#### new 绑定

```javascript
function Foo(name) {
    this.name = name;
}
var foo = new Foo('xyue');
foo.name;  // xyue
```

[new 实现原理](/2020/03/17/what-did-new-do/#实现)


### 箭头函数指向函数所在的所用域

箭头函数中没有 `this` 绑定，在箭头函数中 `this` 始终指向函数所在的所用域。（***箭头函数不能作为构造函数***）


## call、apply、bind 实现

在 `javascript` 中，`call`、`apply`、`bind` 都是为了改变某个函数运行时的上下文（context）而存在的，换句话说，就是为了改变函数体内部 `this` 的指向。


- 相同点
   -  均可以改变 `this` 指向，第一个参数为 `this`的指向对象

- 不同点
  - 函数调用 `call`、`apply` 时，直接调用函数，而 `bind` 返回一个新的函数
  -  `call`、`bind` 接收序列参数，`apply` 接收参数数组

### call 

举个例子

```javascript 
var foo = {
    value: 1
};
function bar() {
    console.log(this.value);
}
bar.call(foo); // 1
```

注意两点：
 1. `call` 改变了 `this` 的指向，指向到 `foo`
 2. `bar` 函数执行了

那么我们该怎么实现这两个效果呢？

试想当调用 `call` 的时候，把 `foo` 对象改造成如下：

```javascript 
var foo = {
    value: 1,
    bar: function() {
        console.log(this.value)
    }
};
foo.bar(); // 1
```

这个时候 `this` 就指向了 `foo`。

所以我们实现的步骤可以分为：

1. 将函数设为对象的属性
2. 执行该函数
3. 删除该函数

```javascript
Function.prototype.myCall = function(context) {
    var context = context || window;
    context.fn = this;
    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push(arguments[i]);
    }
    var result;
    if(args.length > 0){
        result = eval('context.fn(' + args +')');
    }else {
        result = context.fn();
    }
    delete context;
    return result;
}
```

或者使用 `ES6` 扩展运算符，解构赋值

```javascript
Function.prototype.myCall = function() {
    var [context, ...args] = arguments;
    var context = context || window;
    context.fn = this;
    var result = context.fn(...args);
    delete context;
    return result;
}
```

### apply

`apply` 的实现跟 `call` 类似，差异在与参数的传递上：

```javascript
Function.prototype.myApply = function() {
    // args 第二个参数为数组
    var [context, args] = arguments;
    var context = context || window;
    context.fn = this;
    var result = context.fn(...args);
    delete context;
    return result;
}
```

### bind 

> `bind()` 方法会创建一个新函数。当这个新函数被调用时，`bind()` 的第一个参数将作为它运行时的 `this`，之后的一序列参数将会在传递的实参前传入作为它的参数。(来自于 MDN )

可以看出 bind 的 2 个特点：

1. 返回一个函数
2. 可以传入参数

```javascript 
Function.prototype.myBind = function(context) {
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);
    return function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(context, args.concat(bindArgs));
    }
}
```

但这样还存在一个问题，如下

```javascript 
var foo = {
    value: 1
};
function Bar(name,age) {
    this.hobby = 'sing';
    console.log(this.value);
    console.log(name);
    console.log(age);
}
Bar.prototype.friend = 'xyue';

var bindFoo1 = Bar.bind(foo, 'fuzy');
var obj1 = new bindFoo1('18');
// undefined
// fuzy
// 18
console.log(obj1.hobby); // sing
console.log(obj1.friend); // xyue

var bindFoo2 = Bar.myBind(foo, 'fuzy');
var obj2 = new bindFoo2('18');
// 1
// fuzy
// 18
console.log(obj2.hobby); // undefined
console.log(obj2.friend); // undefined
```

>一个绑定函数也能使用 `new` 操作符创建对象：这种行为就像把原函数当成构造器。提供的 `this` 值被忽略，同时调用时的参数被提供给模拟函数。

我们看到，当 `bind` 返回的函数作为构造函数的时候，`bind` 时指定的 `this` 值会失效，但传入的参数依然生效。

所以我们可以通过修改返回的函数的原型来实现：

```javascript
Function.prototype.myBind = function(context) {
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);
    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
    };
    fBound.prototype = this.prototype;
    return fBound;
}
```

但是在这个写法中，我们直接将 fBound.prototype = this.prototype，我们直接修改 fBound.prototype 的时候，也会直接修改绑定函数的 prototype。
这个时候，我们可以通过一个空函数来进行中转：

```javascript
Function.prototype.myBind = function(context) {
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);
    var fNOP = function () {};
    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    };
    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```

说到这，顺便说一下 `Object.create`，该方法创建一个新对象，使用现有的对象来提供新创建的对象的 `__proto__`。
不考虑第二个参数，对其主要功能做简单实现：

```javascript
Object.create = function(obj) {
    function f() {};
    f.prototype = obj;
    return new f;
};
```

可以看到我们可以发现，这不就是创建一个空函数进行中转么，所以我们的实现代码如下：

```javascript
Function.prototype.myBind = function(context) {
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);
    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
    };
    fBound.prototype = Object.create(this.prototype);
    return fBound;
}
```

当然这都是模拟实现，并不是完美的，比如：

```javascript
var foo = {
    value: 1
};
function Bar(name,age) {
    this.hobby = 'sing';
    console.log(this.value);
    console.log(name);
    console.log(age);
}
Bar.prototype.friend = 'xyue';

var bindFoo1 = Bar.bind(foo, 'fuzy');
var bindFoo2 = Bar.myBind(foo, 'fuzy');
console.log(bindFoo1.prototype); // undefined
console.log(bindFoo2.prototype); // Bar {}
```

我们看到原生的 `bind` 返回的函数是没有 `prototype` 属性的。
正如原生的 `bind` 方法的特性定义的那样。
> `bind `方法所返回的函数并不包含 `prototype` 属性，并且将这些绑定的函数用作构造函数所创建的对象从原始的未绑定的构造函数中继承 `prototype`

所以我们用 `es6` 方法实现做一些补充：

```javascript
Function.prototype.myBind = function(context, ...args) {
    var self = this;
    var Fn = function(...bindArgs) {
        var allArgs = args.concat(bindArgs);
        if(this instanceof Fn) {
            return new self(...allArgs);
        }
        return self.apply(context, allArgs);
    }
    delete Fn.prototype;
    return Fn;
}
```

这样与原生的 `bind` 表现就大致一样了。