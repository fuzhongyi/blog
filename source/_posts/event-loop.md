---
title: 从多线程到 Event Loop
date: 2020-03-21 13:45:07
tags: javascript
categories: 技术水波文
---

![](/event-loop/header-img.webp)

在我们的认知中，javascript 是单线程的，但它又是如何完成诸如异步请求这种多线程操作的呢？
我们先从进程、线程的角度来了解这个问题。

## 进程与线程

+ `进程` 是 `cpu` 资源分配的最小单位（是能拥有资源和独立运行的最小单位）
+ `线程` 是 `cpu` 调度的最小单位（线程是建立在进程的基础上的一次程序运行单位，一个进程中可以有多个线程）
+ 不同进程之间也可以通信，代价较大

简单理解，我们把 `cpu` 比作工厂，工厂有多个车间，但工厂的电力有限，每次只能供给一个车间使用（单个 `cpu` 一次只能运行一个任务）。`进程` 就好比工厂的车间，它代表 `cpu` 所能处理的单个任务, 进程之间相互独立，任一时刻，`cpu` 总是运行一个进程，其他进程处于非运行状态。 `cpu` 使用时间片轮转进度算法来实现同时运行多个进程。`线程` 就好比车间里的工人，一个车间里，可以有很多工人，共享车间所有的资源，他们协同完成一个任务。

一个 `进程` 可以包括多个 `线程`，多个 `线程` 共享 `进程` 资源。

## 浏览器包含了哪些进程

+ 主进程
  + 协调控制其他子进程（创建、销毁）
  + 浏览器界面显示，用户交互，前进、后退、收藏
  + 将渲染进程得到的内存中的 Bitmap，绘制到用户界面上
  + 处理不可见操作，网络请求，文件访问等

+ 第三方插件进程

  + 每种类型的插件对应一个进程，仅当使用该插件时才创建
+ GPU进程
  + 用于3D绘制等

+ 渲染进程，就是我们说的浏览器内核
  + 负责页面渲染，脚本执行，事件处理等
  + 每个tab页一个渲染进程

了解到浏览器所包含的进程，我们可以发现，渲染进程是前端操作最重要的一环。

## 渲染进程

我们知道，进程和线程是一对多的关系，也就是说一个进程包含了多条线程。

对于 `渲染进程` 而言，它也是多线程的，包含：

+ GUI 渲染线程
  + 负责渲染页面，布局和绘制
  + 页面需要重绘和回流时，该线程就会执行
  + 与 JS 引擎线程互斥，防止渲染结果不可预期

+ JS 引擎线程
  + 负责处理解析和执行 javascript 脚本程序
  + 只有一个 JS 引擎线程（单线程）
  + 与 GUI 渲染线程互斥，防止渲染结果不可预期

+ 事件触发线程
  + 用来控制事件循环（鼠标点击、setTimeout、ajax 等）
  + 当事件满足触发条件时，将事件放入到JS引擎所在的执行队列中

+ 定时触发器线程
  + setInterval 与 setTimeout 所在的线程
  + 定时任务并不是由 JS 引擎计时的，是由定时触发线程来计时的
  + 计时完毕后，通知事件触发线程

+ 异步 http 请求线程
  + 浏览器有一个单独的线程用于处理 ajax 请求
  + 当请求完成时，若有回调函数，通知事件触发线程

当我们了解了渲染进程包含的线程后，我们思考为什么 javascript 是单线程的。

## 为什么是单线程

因为多线程的复杂性，多线程操作需要加锁，编码的复杂性会增高。如果同时操作 DOM ，在多线程不加锁的情况下，最终会导致 DOM 渲染的结果不可预期。

## 从 event loop 看 JS 运行机制

到了这里，终于要进入我们的主题，什么是 `event loop`。

先理解一些概念：

+ JS 分为同步任务和异步任务
+ 同步任务都在 JS 引擎线程上执行，形成一个执行栈
+ 事件触发线程管理一个任务队列，异步任务触发条件达成，将回调事件放到任务队列中
+ 执行栈中所有同步任务执行完毕，此时 JS 引擎线程空闲，系统会读取任务队列，将可运行的异步任务回调事件添加到执行栈中，开始执行

![](/event-loop/5-1.png)

在前端开发中我们会通过 `setTimeout`、`setInterval` 来指定定时任务，通过 `xhr`、`fetch` 发送网络请求。

我们知道，不管是这些定时任务或者网络请求，这些代码在执行，本身是同步任务的，而其中的回调函数才是异步任务。

当代码执行到 `setTimeout`、`setInterval` 时，实际上是 `JS引擎线程` 通知 `定时触发器线程`，在间隔指定时间后，会触发一个回调事件， 而 `定时触发器线程` 在接收到这个消息后，会在等待的时间后，将回调事件放入到由 `事件触发线程` 所管理的事件队列中。

当代码执行到 `xhr`、`fetch` 时，实际上是 `JS引擎线程` 通知 `异步http请求线程`，发送一个网络请求，并制定请求完成后的回调事件， 而 `异步http请求线程` 在接收到这个消息后，会在请求成功后，将回调事件放入到由`事件触发线程` 所管理的事件队列中。

当我们的同步任务执行完，`JS引擎线程` 会询问 `事件触发线程`，在事件队列中是否有待执行的回调函数，如果有就会加入到执行栈中交给 `JS引擎线程` 执行。

用一张图来解释：

![](/event-loop/5-2.webp)

总结：

1. JS 引擎线程执行执行栈中的事件
2. 执行栈中的事件执行完毕，就开始读取事件队列中的事件
3. 事件队列中的回调事件加入到执行栈依次执行

`2`,`3` 重复循环

## 宏任务、微任务

当我们基本了解了什么是执行栈，什么是事件队列之后，我们深入了解一下事件循环中 `宏任务`、`微任务`。

### 宏任务

我们可以将每次执行栈执行的代码当做是一个宏任务（包括每次从事件队列中获取一个事件回调并放到执行栈中执行），
每一个宏任务会从头到尾执行完毕，不会执行其他。
我们前文提到过 `JS引擎线程` 和 `GUI渲染线程` 是互斥的关系，浏览器为了能够使宏任务和 `DOM` 任务有序的进行，会在一个宏任务执行结果后，在下一个宏任务执行前，`GUI 渲染线程` 开始工作，对页面进行渲染。

```javascript
// 宏任务-->渲染-->宏任务-->渲染-->．．．
```

`script（主代码块）`，`setTimeout`，`setInterval`、`I/O(ajax)`,`setImmedidate(node)`，都属于宏任务。

举个例子🌰：

```javascript
document.body.style = 'background:black';
document.body.style = 'background:red';
document.body.style = 'background:blue';
document.body.style = 'background:grey';
```

我们将这段代码放到浏览器控制台执行一下，看一下效果：

![](/event-loop/6-1.gif)

我们看到，页面背景会在瞬间变成灰色。这是因为以上代码属于同一次宏任务，所以全部执行完才触发页面渲染，渲染时`GUI 渲染线程`会将所有 UI 改动合并优化，所以在视觉效果上，只会看到页面变成灰色。

再来个例子🌰🌰：

```javascript
document.body.style = 'background:blue';
setTimeout(function() {
    document.body.style = 'background:black'
}, 0);
```

执行一下，看下效果：

![](/event-loop/6-2.gif)

我们看到，页面先显示成蓝色背景，然后又变成了黑色背景，这是因为以上代码属于两次 `宏任务`，第一次 `宏任务` 执行的代码是将背景变成蓝色，然后触发渲染，将页面变成蓝色，再触发第二次 `宏任务` 将背景变成黑色。

### 微任务

我们已经知道 `宏任务` 结束后，会执行渲染，然后执行下一个 `宏任务`， 而 `微任务` 可以理解成在当前 `宏任务` 执行后立即执行的任务。

也就是说，当 `宏任务` 执行完，会在渲染前，将执行期间所产生的所有 `微任务` 都执行完。

```javascript
// 宏任务-->微任务-->渲染-->宏任务-->微任务-->渲染-->．．．
```

`process.nextTick(node)`，`Promise`，`MutationObserver` 属于微任务。

`process.nextTick` 是有一个插队操作的，就是说他进入微任务队列时，会插到除了 `process.nextTick` 其他的微任务前面。

举个例子🌰：

```javascript
document.body.style = 'background:blue'
console.log(1);
Promise.resolve().then(()=>{
    console.log(2);
    document.body.style = 'background:black'
});
console.log(3);
```

执行一下，看下效果：

![](/event-loop/6-3.gif)

控制台输出 1 3 2 , 是因为 `promise` 对象的 `then` 方法的回调函数是异步执行，所以 2 最后输出页面的背景色直接变成黑色，没有经过蓝色的阶段，是因为，我们在宏任务中将背景设置为蓝色，但在进行渲染前执行了微任务，在微任务中将背景变成了黑色，然后才执行的渲染。

再比如说：

```javascript
setTimeout(() => {
    console.log(1);
    Promise.resolve(3).then(data => console.log(data));
}, 0);
setTimeout(() => {
    console.log(2)
}, 0);
// 1 3 2
```

上面代码共包含两个 `setTimeout`，也就是说除主代码块外，还有两个 `宏任务`，其中第一个 `宏任务` 执行中，输出 1，并且创建了 `微任务队列`，所以在下一个 `宏任务队列` 执行前，先执行 `微任务`，在 `微任务` 执行中，输出 3 ，`微任务` 执行后，执行下一次 `宏任务`，执行中输出 2。

## 总结

+ 执行一个`宏任务` （栈中没有就从 `事件队列` 中获取）
+ 执行过程中如果遇到 `微任务`，就将它添加到 `微任务队列`中
+ 宏任务执行完毕后，立即执行当前 `微任务队列` 中的所有 `微任务` （依次执行）
+ 当前 `宏任务` 执行完毕，开始检查渲染，然后 `GUI 渲染线程` 接管渲染
+ 渲染完毕后，`JS引擎线程` 继续接管，开始下一个 `宏任务`（从事件队列中获取）

![](/event-loop/7-1.webp)

## 题外话

``` javascript
setTimeout(()=>console.log(3), 2);
setTimeout(()=>console.log(2), 1);
setTimeout(()=>console.log(1), 0);
```

没有深入接触过 `timer` 的同学如果直接从代码中的延时设置来看，会回答：`1、2、3`。

而另一些有一定经验的同学可能会回答：`3、2、1`。因为 [MDN 的 setTimeout](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setTimeout) 文档中提到 `HTML` 规范最低延时为 **4ms**：

*(补充说明：最低延时的设置是为了给CPU留下休息时间）*

> In fact, 4ms is specified by the HTML5 spec and is consistent across browsers released in 2010 and onward. Prior to (Firefox 5.0 / Thunderbird 5.0 / SeaMonkey 2.2), the minimum timeout value for nested timeouts was 10 ms.

而真正痛过的同学会告诉你，答案是：`1、0、2`。

我们发现 0ms 和 1ms 的延时效果是一样的，通过资料我们发现：

### 在 chrome 中

```javascript
// https://chromium.googlesource.com/chromium/blink/+/master/Source/core/frame/DOMTimer.cpp#93
double intervalMilliseconds = std::max(oneMillisecond, interval * oneMillisecond);
```

这里 interval 就是传入的数值，可以看出传入 0 和传入 1 结果都是 oneMillisecond，即 1ms。

这样解释了为何 1ms 和 0ms 行为是一致的，那 4ms 到底是怎么回事？查阅 [HTML 规范](https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#timers)，发现虽然有 4ms 的限制，但是是存在条件的，详见规范第 11 点：

> If nesting level is greater than 5, and timeout is less than 4, then set timeout to 4.

### 在 node 中

```javascript
// https://github.com/nodejs/node/blob/v8.9.4/lib/timers.js#L456
if (!(after >= 1 && after <= TIMEOUT_MAX))
  after = 1; // schedule on next tick, follows browser behavior
```

代码中的注释直接说明了，设置最低 1ms 的行为是为了向浏览器行为看齐。
