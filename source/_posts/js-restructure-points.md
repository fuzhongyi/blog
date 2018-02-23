---
title: js 代码重构细节
date: 2017-12-06 15:30:41
tags: [javascript,重构]
categories: 读书笔记
---

![](/js-restructure-points/header-img.jpg)

>重构（名词）：对软件内部结构的一种调整，目的是在不改变软件可观察行为的前提下，提高其可理解性，降低其修改成本。

伴随软件的迭代，功能的日益复杂，由于没有理解原来程序的结构和考虑代码的扩展性，程序慢慢的失去了原来的结构，之前的设计变得面目全非。如果不进行重构，程序就会越来越难以维护。

## 为何重构

1. 改进软件设计 *—— 原来的设计没考虑到一些未知的情况，导致其后加入的功能破坏原有设计*
2. 使软件更容易理解
3. 帮助找到bug *—— 重构可以增加对代码的理解，从而更容易*
4. 提高编程速度 *—— 重构虽然花费时间，但重构可以改善程序的设计，使添加新特性更容易*

## 何时重构

重构应该随时进行，当加入新功能时不是特别容易时，可以通过重构使添加新特性更容易，修补错误时更容易发现Bug，复审代码也是重构的好时机。

## 重构细节

在实际的项目重构有一些常见而容易忽略的细节，这些细节也是帮助我们达到重构目标的重要手段。有一部分思想来自 **Martin Fowler** 的名著 [《重构：改善既有代码的设计》](http://p02hf9fn0.bkt.clouddn.com/重构：改善既有代码的设计.pdf)，虽然该书是使用 `Java` 语言写成的，但这些重构的技巧，有很大一部分可以为 `JavaScript` 语言所借鉴。

### 提炼函数

如果在函数中有一段代码可以被独立出来，那我们最好把这些代码放进另外一个独立的函数中。这是一种很常见的优化工作，这样做的好处主要有以下几点。

+ 避免出现超大函数
+ 独立出来的函数有助于代码复用
+ 独立出来的函数更容易被覆写
+ 独立出来的函数如果拥有一个良好的命名，它本身就起到了注释的作用

```javascript
var getUserInfo = function () {
    ajax('http:// xxx.com/userInfo', function (data) {
        console.log('userId: ' + data.userId);
        console.log('userName: ' + data.userName);
        console.log('nickName: ' + data.nickName);
    });
};
```

更改为↓

```javascript
var getUserInfo = function () {
    ajax('http:// xxx.com/userInfo', function (data) {
        printDetails(data);
    });
};
var printDetails = function (data) {
    console.log('userId: ' + data.userId);
    console.log('userName: ' + data.userName);
    console.log('nickName: ' + data.nickName);
};
```

### 合并重复的条件片段

```javascript
var paging = function (currPage) {
    if (currPage <= 0) {
        currPage = 0;
        jump(currPage); // 跳转
    } else if (currPage >= totalPage) {
        currPage = totalPage;
        jump(currPage); // 跳转
    } else {
        jump(currPage); // 跳转
    }
};
```

更改为↓

```javascript
var paging = function (currPage) {
    if (currPage <= 0) {
        currPage = 0;
    } else if (currPage >= totalPage) {
        currPage = totalPage;
    }
    jump(currPage); // 把 jump 函数独立出来
};
```

### 把条件分支语句提炼成函数

```javascript
var getPrice = function (price) {
    var date = new Date();
    if (date.getMonth() >= 6 && date.getMonth() <= 9) { // 夏天
        return price * 0.8;
    }
    return price;
};
```

更改为↓

```javascript
var isSummer = function () {
    var date = new Date();
    return date.getMonth() >= 6 && date.getMonth() <= 9;
};
var getPrice = function (price) {
    if (isSummer()) { // 夏天
        return price * 0.8;
    }
    return price;
};
```

### 合理使用循环

```javascript
var createXHR = function () {
    var xhr;
    try {
        xhr = new ActiveXObject('MSXML2.XMLHttp.6.0');
    } catch (e) {
        try {
            xhr = new ActiveXObject('MSXML2.XMLHttp.3.0');
        } catch (e) {
            xhr = new ActiveXObject('MSXML2.XMLHttp');
        }
    }
    return xhr;
};
```

更改为↓

```javascript
var createXHR = function () {
    var versions = ['MSXML2.XMLHttp.6.0ddd', 'MSXML2.XMLHttp.3.0', 'MSXML2.XMLHttp'];
    for (var i = 0, version; version = versions[i++];) {
        try {
            return new ActiveXObject(version);
        } catch (e) {
        }
    }
};
```

### 提前让函数退出代替嵌套条件分支

```javascript
var del = function (obj) {
    var ret;
    if (!obj.isReadOnly) { // 不为只读的才能被删除
        if (obj.isFolder) { // 如果是文件夹
            ret = deleteFolder(obj);
        } else if (obj.isFile) { // 如果是文件
            ret = deleteFile(obj);
        }
    }
    return ret;
};
```

更改为↓

```javascript
var del = function (obj) {
    if (obj.isReadOnly) { // 反转 if 表达式
        return;
    }
    if (obj.isFolder) {
        return deleteFolder(obj);
    }
    if (obj.isFile) {
        return deleteFile(obj);
    }
};
```

### 传递对象参数代替过长的参数列表

```javascript
var setUserInfo = function (id, name, address, sex, mobile, qq) {
    console.log('id= ' + id);
    console.log('name= ' + name);
    console.log('address= ' + address);
    console.log('sex= ' + sex);
    console.log('mobile= ' + mobile);
    console.log('qq= ' + qq);
};
```

更改为↓

```javascript
var setUserInfo = function (obj) {
    console.log('id= ' + obj.id);
    console.log('name= ' + obj.name);
    console.log('address= ' + obj.address);
    console.log('sex= ' + obj.sex);
    console.log('mobile= ' + obj.mobile);
    console.log('qq= ' + obj.qq);
};
```

### 尽量减少参数数量

```javascript
var draw = function (width, height, square) {};
```

更改为↓

```javascript
var draw = function (width, height) {
    var square = width * height;
};
```

### 少用三目运算符


如果条件分支逻辑简单且清晰，这无碍我们使用三目运算符：
```javascript
var global = typeof window !== "undefined" ? window : this;
```

但如果条件分支逻辑非常复杂，如下段代码所示，那我们最好的选择还是按部就班地编写 `if`、 `else`。 `if`、 `else` 语句的好处很多，一是阅读相对容易，二是修改的时候比修改三目运算符周围的代码更加方便

```javascript
if ( !aup || !bup ) {   // 头痛吧
    return a === doc ? -1 :
        b === doc ? 1 :
        aup ? -1 :
        bup ? 1 :
        sortInput ?
        ( indexOf.call( sortInput, a ) - indexOf.call( sortInput, b ) ) :
        0;
}
```
### 合理使用链式调用

```javascript
var User = function () {
    this.id = null;
    this.name = null;
};
User.prototype.setId = function (id) {
    this.id = id;
    return this;
};
User.prototype.setName = function (name) {
    this.name = name;
    return this;
};
console.log(new User().setId(1314).setName('sven'));
```

或则↓

```javascript
var User = {
    id: null,
    name: null,
    setId: function (id) {
        this.id = id;
        return this;
    },
    setName: function (name) {
        this.name = name;
        return this;
    }
};
console.log(User.setId(1314).setName('sven'));
```

### 分解大型类

```javascript
var Spirit = function (name) {
    this.name = name;
};
Spirit.prototype.attack = function (type) { // 攻击
    if (type === 'waveBoxing') {
        console.log(this.name + ': 使用波动拳');
    } else if (type === 'whirlKick') {
        console.log(this.name + ': 使用旋风腿');
    }
};
var spirit = new Spirit('RYU');
spirit.attack('waveBoxing'); // 输出： RYU: 使用波动拳
spirit.attack('whirlKick');  // 输出： RYU: 使用旋风腿
```

更改为↓

```javascript
var Attack = function (spirit) {
    this.spirit = spirit;
};
Attack.prototype.start = function (type) {
    return this.list[type].call(this);
};
Attack.prototype.list = {
    waveBoxing: function () {
        console.log(this.spirit.name + ': 使用波动拳');
    },
    whirlKick: function () {
        console.log(this.spirit.name + ': 使用旋风腿');
    }
};
var Spirit = function (name) {
    this.name = name;
    this.attackObj = new Attack(this);
};
Spirit.prototype.attack = function (type) { // 攻击
    this.attackObj.start(type);
};
var spirit = new Spirit('RYU');
spirit.attack('waveBoxing');  // 输出： RYU: 使用波动拳
spirit.attack('whirlKick');   // 输出： RYU: 使用旋风腿
```

### 用 return 退出多重循环

```javascript
var func = function () {
    var flag = false;
    for (var i = 0; i < 10; i++) {
        for (var j = 0; j < 10; j++) {
            if (i * j > 30) {
                flag = true;
                break;
            }
        }
        if (flag === true) {
            break;
        }
    }
    console.log('....');
};
```

更改为↓

```javascript
var func = function () {
    for (var i = 0; i < 10; i++) {
        for (var j = 0; j < 10; j++) {
            if (i * j > 30) { 
                console.log('....');  // for 循环后边的代码提取到 return 之前执行
                return;    
            }
        }
    }
};
```

## References

[《JavaScript设计模式与开发实践》——曾探](http://p02hf9fn0.bkt.clouddn.com/JavaScript设计模式与开发实践.pdf.pdf)