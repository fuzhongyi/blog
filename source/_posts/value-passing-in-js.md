---
title: JS中的值传递
date: 2017-09-12 10:12:17
tags: javascript
categories: 技术水波文
---

![](/value-passing-in-js/header-img.jpg)

## 值传递?
相信在看到这个标题时，很多童鞋都会有这样的疑惑——“What?不是还有引用传递么”。
像这样：
```javascript
var person = { name : 'lilei' };
function setName (obj) {
    obj.name = 'hanmeimei';
    console.log(obj.name);  //output: hanmeimei
}
setName(person);
console.log(person.name);   //output: hanmeimei
```
确实，对象person属性name的值被改变了，最初的我对此也是深信不疑。
在《JavaScript高级程序设计》第三版 4.1.3中，关于参数的传递这样讲到：
>ECMAScript中所有函数的参数都是按值传递的。

啥子哎！连红宝书都这样说，那可得好好深究下。

## 定义
在解开疑惑前，我们先了解下什么是按值传递(call by value)，什么是按引用传递(call by reference)。
在计算机科学里，这个部分叫求值策略(Evaluation Strategy)。它决定变量之间、函数调用时实参和形参之间值是如何传递的。

### 传递方式
+ 值传递(call by value)
常用的求值策略，函数的形参是被调用时所传实参的副本。修改形参的值并不会影响实参。

+ 引用传递(call by reference)
函数的形参接收实参的隐式引用，而不再是副本。这意味着函数形参的值如果被修改，实参也会被修改。同时两者指向相同的值。

### 数据类型
ECMAScript包括两个不同类型的值：基本数据类型和引用数据类型。

+ 基本数据类型：
保存在栈内存中的简单数据段，常见的基本数据类型 `Number`、`String` 、`Boolean`、`Null`和`Undefined`，最新的ECMAScript标准定义加入了`Symbol`。

+ 引用数据类型：
保存在堆内存中的由多个值构成的对象，比如：`Array`、`Function`和`Object`，从底层技术上看，它们三都是对象。
与其他语言的不同是，你不可以直接访问堆内存空间中的位置和操作堆内存空间。只能操作对象在栈内存中的引用地址。

## 引用传递?

如果一个基本数据类型绑定给某个变量，我们可以认为该变量包含这个基本数据类型的值。
```javascript
var name = 'lilei';
function setName (n) {
    n = 'hanmeimei';
    console.log(n);     //output: hanmeimei
}
setName(name);
console.log(name);      //output: lilei
```
当我们将新值重新赋给变量，可以看到这里 name 值并没有改变,实际上 n 只是保存了 name 复制的一个副本(函数的形参是被调用时所传实参的副本)。

|变量|值| 
|---|---|
| name   |lilei|
| n      |lilei|

所以，n 的改变对 name 没有影响。

|变量|值| 
|---|---| 
| name   |lilei |
| n      |hanmeimei|

我们把它称作值传递。
可以这样理解，当传递 name 到函数 setName 中，相当于拷贝了一份 name ，假设拷贝的这份叫 _name ，函数中修改的都是 _name 的值，而不会影响原来的 name 值。

回到最前边我们看到的那段代码，稍加改动:
```javascript
var person = { name : 'lilei' };
function setName (obj) {
    // obj.name = 'hanmeimei';
    obj = { name : 'hanmeimei' };
    console.log(obj.name);  //output: hanmeimei
}
setName(person);
console.log(person.name);   //output: lilei
```
所谓按引用传递，就是传递对象的引用，函数内部对参数的任何改变都会影响该对象的值，因为两者引用的是同一个对象。
但这里 obj 的改变并没有影响 person，这究竟是引用传递吗？

## 解惑

看到这，相信你对之前的理解已有了不一样的认识。
既然这不是引用传递，还会是值传递么？

基本数据类型传递值，引用数据类型传递引用地址(**引用类型数据在栈内存中保存的实际上是对象在堆内存中的引用地址。通过这个引用地址可以快速查找到保存中堆内存中的对象**)。

正如前边定义所提到的那样，值传递，**函数的形参是被调用时所传实参的副本**。
不难理解，当 person 

|变量|值|地址|对象|
|---|---|---|---|
|person|<#001>|#001|{ name : 'lilei' }|

做为参数进入函数 setName 后，就有了地址副本，这个地址副本和 person 的地址指向是相同的。

|变量|值|地址|对象|
|---|---|---|---|
|person|<#001>|#001|{ name : 'lilei' }|
|obj|<#001>|#001||
||<#002>|#002|{ name : 'hanmeimei' }|

如果这个时候我们对这个副本操作改变其属性值，则指向这个地址的变量都会发生变化。
但我们为 obj 重新赋了值，将地址副本指向改变，指向了新的对象。

|变量|值|地址|对象|
|---|---|---|---|
|person|<#001>|#001|{ name : 'lilei' }|
|obj|<#002>|#002|{ name : 'hanmeimei' }|

这样一来 obj 和 person 就完全断了， obj 的改变并不会影响 person。 

**传递对象引用的副本**，这样的传递方式又被称之为共享传递(call by sharing)。
拷贝副本，也是一种值的拷贝，所以在JS中只有值传递。😉