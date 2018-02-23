---
title: JS 易混淆的方法整理
date: 2018-02-23 14:12:58
tags: javascript
categories: [备忘,转载]
---

>js的字符串方法如`substr`、`substring`，数组方法如`slice`、`splice`等名字相近，用法稍有不同，让开发者在开发过程中总是记不起其用法，需要查资料。现整理一下，希望有助大家记忆。

## String 对象

### slice
>stringObject.slice(start, end)

```javascript
var a = 'Hello world!';
var b = a.slice(2);
var c = a.slice(-4, -2);
// a: 'Hello world!'
// b: 'llo world!'
// c: 'rl'，参数可为负
```

### substr

>stringObject.substr(start, length)

```javascript
var a = 'Hello world!';
var b = a.substr(0, 4);
var c = a.substr(-5, 2);
// a: 'Hello world!'
// b: 'Hell'
// c: 'or'，参数可为负
```

### substring

>stringObject.substring(start, stop)

```javascript
var a = 'Hello world!';
var b = a.substring(0, 4);
var c = a.substring(3, 2);
var d = a.substring(0, -1);
// a: 'Hello world!'
// b: 'Hell'
// c: 'l'，start比stop小，交换这两个参数
// d: ''，参数为负，返回空字符串
```

>slice、substr、substring都是字符串的切割方法，三者之间有细微的区别，根据不同的使用场景可以灵活使用。三种方法都是生成新的字符串，而不是修改原string。

## Array对象

### concat

>arrayObject.concat(arrayX, arrayX, ..., arrayX)

参数可以为具体的值，也可以为数组对象，可以任意多个。不改变现有的数组，返回被连接数组的一个副本。

```javascript
var a = [1, 2, 3];
var b = a.concat(4, 5);
var c = a.concat([4, 5]);
// a: [1, 2, 3]
// b: [1, 2, 3, 4, 5]
// c: [1, 2, 3, 4, 5]
```

### pop

>arrayObject.pop()

删除 arrayObject 的最后一个元素，把数组长度减 1，并且返回它删除的元素的值。如果数组已经为空，则 pop() 不改变数组，并返回 undefined 值。该方法会改变原数组。

```javascript
var a = [1, 2, 3];
var b = a.pop();
// a: [1, 2]，修改了原数组
// b: 3，返回删除元素的值
```

### push

>arrayObject.push(newelement1,newelement2,...,newelementX)\

参数顺序添加到 arrayObject 的尾部，直接修改 arrayObject。

```javascript
var a = [1, 2, 3];
var b = a.push(4, 5);
// a: [1, 2, 3, 4, 5]，修改了原数组
// b: 5，返回修改后的数组的长度
```

### shift

>arrayObject.shift()

把数组的第一个元素从其中删除，并返回第一个元素的值。如果数组是空的，那么 shift() 方法将不进行任何操作，返回 undefined 值。该方法会改变原数组。类比pop方法。

```javascript
var a = [1, 2, 3];
var b = a.shift();
// a: [2, 3]，修改了原数组
// b: 1，返回删除元素的值
```

### unshift

>arrayObject.unshift(newelement1, newelement2, ...., newelementX)

向数组的开头添加一个或更多元素，并返回新的长度。该方法的第一个参数将成为数组的新元素 0，如果还有第二个参数，它将成为新的元素 1，以此类推。

```javascript
var a = [1, 2, 3];
var b = a.unshift(4, 5);
// a: [ 4, 5, 1, 2, 3 ]，修改了原数组
// b: 5，返回修改后的数组的长度
```

### slice

>arrayObject.slice(start, end)

返回一个新的数组，包含从 start 到 end （不包括该元素）的 arrayObject 中的元素。该方法不会修改原数组。

```javascript
var a = [1, 2, 3, 4, 5];
var b = a.slice(2);
// a: [1, 2, 3, 4, 5]，不修改原数组
// b: [3, 4, 5]，返回新数组

var c = [1, 2, 3, 4, 5];
var d = c.slice(2, -1);
// c: [1, 2, 3, 4, 5]，不修改原数组
// d: [3, 4]，返回新数组
```

### splice

>arrayObject.splice(index, howmany, item1, ..., itemX)

可删除从 index 处开始的零个或多个元素，并且用参数列表中声明的一个或多个值来替换那些被删除的元素。如果从 arrayObject 中删除了元素，则返回的是含有被删除的元素的数组。

```javascript
var a = [1, 2, 3, 4, 5];
var b = a.splice(1, 1);
// a: [1, 3, 4, 5]，修改了原数组
// b: [2]，返回新数组

var c = [1, 2, 3, 4, 5];
var d = c.splice(-1, 1);
// c: [1, 2, 3, 4]，修改了原数组
// d: [5]，返回新数组

var e = [1, 2, 3, 4, 5];
var f = e.splice(1, 1, 6, 7);
// e: [ 1, 6, 7, 3, 4, 5 ]，修改了原数组
// f: [2]，返回新数组

var g = [1, 2, 3, 4, 5];
var h = g.splice(1, 0, 8);
// g: [ 1, 8, 2, 3, 4, 5 ]，修改了原数组
// h: []，没有删除值，返回空数组
```

### sort

>arrayObject.sort(sortBy)

无参数时，将按字母顺序对数组中的元素进行排序。参数为比较函数时，如果要交换 prev 和 next 的值，返回大于 0 的值。

```javascript
var a = [1, 10, 8, 6, 9];
var b = a.sort(function (prev, next) {
  return prev - next;
});
// a: [1, 6, 8, 9, 10]，修改了原数组
// b: [1, 6, 8, 9, 10]，返回修改后的数组
```

### reverse

>arrayObject.reverse()

用于颠倒数组中元素的顺序。会改变原数组。

```javascript
var a = [1, 2, 3];
var b = a.reverse();
// a: [3, 2, 1]，修改了原数组
// b: [3, 2, 1]，返回修改后的数组
```

### map

>arrayObject.map(function(currentValue, index, arrayObject) {})

对数组的每一项进行处理，返回新数组。

```javascript
var a = [1, 2, 3];
var b = a.map((curVal) => curVal * 2);
// a: [1, 2, 3]，不修改原数组
// b: [2, 4, 6]，返回新数组
```

### forEach

>arrayObject.forEach(function(currentValue, index, arrayObject) {})

数组的每个元素执行一次提供的函数。一般来说不修改原数组，但也可以通过处理函数修改原数组。该方法很灵活，可类比`for...of`。

```javascript
var a = [1, 2, 3];
var sum = 0;
var b = a.forEach((curVal) => sum += curVal);
// a: [1, 2, 3]，不修改原数组
// b: undefined，forEach不返回值
// sum: 6
```

### find

>arrayObject.find(function(currentValue, index, arrayObject) {})

返回数组中第一个满足测试条件（返回 true ）的元素。如果不存在这样的元素，返回undefined。findIndex类似，只不过返回的是第一个满足测试条件的元素的index。

```javascript
var a = [1, 2, 3];
var b = a.find((curVal) => curVal === 1);
var c = a.find((curVal) => curVal === 4);
// a: [1, 2, 3]，不修改原数组
// b: 1
// c: undefined
```

### filter

>arrayObject.filter(function(currentValue, index, arrayObject) {})

返回数组中所有满足测试条件（返回 true ）的元素组成的数组。如果不存在这样的元素，返回[]。

```javascript
var a = [1, 2, 3];
var b = a.filter((curVal) => curVal > 1);
var c = a.filter((curVal) => curVal > 3);
// a: [1, 2, 3]，不修改原数组
// b: [2, 3]
// c: []
```

### reduce

>arrayObject.filter(function(previousValue, currentValue, currentIndex, arrayObject) {}, initialValue)

接收一个函数作为累加器（accumulator），数组中的每个值（从左到右）开始缩减，最终为一个值。

如果没有 initialValue 参数, reduce 从 index 为 1 开始执行回调函数,跳过第一个 index。如果有 initialValue 参数, reduce 将从 index 为 0 开始执行回调。如果数组是空的并且没有 initialValue 参数,将会抛出 TypeError 错误。如果数组只有一个元素并且没有初始值 initialValue ,或者有 initialValue 但数组是空的, 这个唯一的值直接被返回而不会调用回调函数。

```javascript
var a = [1, 2, 3];
var b = a.reduce((prevResult, curItem) => prevResult + curItem);
// a: [1, 2, 3]，不修改原数组
// b: 6
```

>除了 Array 的`pop`、`push`、`shift`、`unshift`、`splice`、`sort`、`reverse` 这 7 个方法会修改原数组，其他方法均不会修改原数组。

[查看原文](https://juejin.im/post/5a8e3b08f265da4e9449c50e)