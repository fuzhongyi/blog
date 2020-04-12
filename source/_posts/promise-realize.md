---
title: Promise 实现原理
date: 2020-03-30 18:14:44
tags: [javascript, promise]
categories : 技术水波文
---

## 什么是 Promise

在 `javascript` 的世界中，代码都是 `单线程` 执行的。这就导致我们的代码里会有嵌套回调函数，一旦嵌套过多，导致代码不容易理解和维护。为了降低异步编程的复杂性，开发人员一直在寻找各种解决方案，`Promise` 就是用来处理异步操作的其中一个方案。

下面将根据 [Promise/A+](https://promisesaplus.com/) 的规范，来写出符合规范的源码。

## Promise 的基本使用

```javascript
const promise = new Promise((resolve, reject) => {
  resolve('success'); // 这里如果是 reject('fail')
});
promise.then((res)=>{
  console.log(res); // 输出：success
},(err)=>{
  console.log(err); // 上面如果执行 reject('fail')，这里就输出 fail
});
```

resolve 找 then 里的成功回调，reject 找 then 里失败的回调。

接下来我们将按自己的方式一步步实现这样的一个 Promise 类，直接上代码：

```javascript
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

class Promise {
  constructor(executor) {
    this.state = PENDING;
    this.onFulfilled = []; // 成功回调队列
    this.onRejected = []; // 失败回调队列
    // PromiseA+ 2.1
    const resolve = value => {
      if (this.state === PENDING) {
        this.state = FULFILLED;
        this.value = value;
        this.onFulfilled.forEach(fn => fn()); // PromiseA+ 2.2.6.1
      }
    };

    const reject = reason => {
      if (this.state === PENDING) {
        this.state = REJECTED;
        this.reason = reason;
        this.onRejected.forEach(fn => fn()); // PromiseA+ 2.2.6.1
      }
    };

    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }

  then(onFulfilled, onRejected) {
    if (this.state === PENDING) {
      this.onFulfilled.push(() => onFulfilled(this.value));
      this.onRejected.push(() => onRejected(this.reason));
    }
  }
}

module.exports = Promise;
```

这样我们就简单的实现了一个 Promise 应该拥有的基本功能。

## 链式调用

在上述代码中我们并没有处理 then 的链式调用，直接使用则会报错：TypeError: Cannot read property 'then' of undefined。

在 [Promise/A+](https://promisesaplus.com/) 规范中提到：

> 2.2.6 `then` may be called multiple times on the same promise.
2.2.6.1 If/when `promise` is fulfilled, all respective `onFulfilled` callbacks must execute in the order of their originating calls to `then`.
2.2.6.2 If/when `promise` is rejected, all respective `onRejected` callbacks must execute in the order of their originating calls to `then`.
2.2.7 `then` must return a promise [3.3].
2.2.7.1 If either `onFulfilled` or `onRejected` returns a value `x`, run the Promise Resolution Procedure `\[\[Resolve]](promise2, x)`.
2.2.7.2 If either `onFulfilled` or `onRejected` throws an exception `e`, `promise2` must be rejected with e as the reason.
2.2.7.3 If `onFulfilled` is not a function and `promise1` is fulfilled, `promise2` must be fulfilled with the same value as `promise1`.
2.2.7.4 If `onRejected` is not a function and `promise1` is rejected, `promise2` must be rejected with the same reason as `promise1`.

```text
2.2.6 then 方法可能被多次调用
2.2.6.1 如果 promise 变成了 fulfilled 态，所有的 onFulfilled 回调都需要按照 then 的顺序执行
2.2.6.2 如果 promise 变成了 rejected 态，所有的 onRejected 回调都需要按照 then 的顺序执行

2.2.7 then 必须返回一个 promise
2.2.7.1 如果 onFulfilled 或 onRejected 执行的结果为 x，调用 handlePromise
2.2.7.2 如果 onFulfilled 或者 onRejected 执行时抛出异常 e，promise2 需要被 reject
2.2.7.3 如果 onFulfilled 不是一个函数，promise2 以 promise1 的 value fulfilled
2.2.7.4 如果 onRejected 不是一个函数，promise2 以 promise1 的 reason rejected
```

所以 then 方法里可能返回一个值或者返回的一个 Promise 实例，我们需要分别处理这两种情况。

```javascript
class Promise {
  // ... 省略如上

  then(onFulfilled, onRejected) {
    // PromiseA+ 2.2.1 / PromiseA+ 2.2.5 / PromiseA+ 2.2.7.3 / PromiseA+ 2.2.7.4
    // 值穿透 promise.then().then().then(res => { console.log(res); })
    onFulfilled = typeof onFulfilled === "function" ? onFulfilled : value => value;
    onRejected = typeof onRejected === "function" ? onRejected : reason => { throw reason; };
    // PromiseA+ 2.2.7 返回一个新的 Promise
    let promise2 = new Promise((resolve, reject) => {
      if (this.state === FULFILLED) {
        // PromiseA+ 2.2.2
        // PromiseA+ 2.2.4 --- setTimeout 模拟异步任务（规范要求）
        setTimeout(() => {
          try {
            // PromiseA+ 2.2.7.1
            let x = onFulfilled(this.value);
            handlePromise(promise2, x, resolve, reject);
          } catch (e) {
            // PromiseA+ 2.2.7.2
            reject(e);
          }
        });
      } else if (this.state === REJECTED) {
        // PromiseA+ 2.2.3
        setTimeout(() => {
          try {
            let x = onRejected(this.reason);
            handlePromise(promise2, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
      } else if (this.state === PENDING) {
        this.onFulfilled.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value);
              handlePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          });
        });
        this.onRejected.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason);
              handlePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          });
        });
      }
    });
    return promise2;
  }
}

function handlePromise(promise2, x, resolve, reject) {
  // PromiseA+ 2.3.1 promise2 是否等于x，判断是否将自己本身返回，抛出 TypeError 错误
  if (promise2 === x) {
    reject(new TypeError('Chaining cycle'));
  }
  // PromiseA+ 2.3.3
  if (x && typeof x === 'object' || typeof x === 'function') {
    let used; // PromiseA+ 2.3.3.3.3 控制 resolve 或 reject 只执行一次（规范要求）
    try {
      // PromiseA+ 2.3.3.1
      let then = x.then;
      // 如果是函数，就认为它是返回新的 promise
      if (typeof then === 'function') {
        // PromiseA+ 2.3.3.1
        then.call(x, (y) => {
          if (used) return;
          used = true;
          handlePromise(promise2, y, resolve, reject);
        }, (r) => {
          // PromiseA+ 2.3.3.2
          if (used) return;
          used = true;
          reject(r);
        });
      } else {
        // PromiseA+ 2.3.3.4 x 是普通值，直接返回
        if (used) return;
        used = true;
        resolve(x);
      }
    } catch (e) {
      // PromiseA+ 2.3.3.2
      if (used) return;
      used = true;
      reject(e);
    }
  } else {
    // PromiseA+ 2.3.4 x 是普通值，直接返回
    resolve(x);
  }
}

module.exports = Promise;
```

这样我们就实现一个完全符合 [Promise/A+](https://promisesaplus.com/) 规范的 `Promise` 了。

让我们安装测试脚本检测下:

```shell
npm install -g promises-aplus-tests
```

在 Promise 实现的代码中添加如下代码：

```javascript
Promise.defer = Promise.deferred = function () {
  let dfd = {};
  dfd.promise = new Promise((resolve, reject) => {
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd;
}
```

对应目录下执行以下命令:

```shell
promises-aplus-tests promise.js
```

promises-aplus-tests 中共有 872 条测试用例，以上代码，可以完美通过所有用例。

## 整体思路

1. new Promise 时，需要传递一个 executor 执行器，执行器立刻执行
2. executor 接受两个参数，分别是 resolve 和 reject
3. promise 只能从 pending 到 rejected, 或者从 pending 到 fulfilled
4. promise 的状态一旦确认，就不会再改变
5. promise 都有 then 方法，then 接收两个参数，分别是 promise 成功的回调 onFulfilled,和 promise 失败的回调 onRejected
6. 如果调用 then 时，promise 已经成功，则执行 onFulfilled，并将 promise 的值作为参数传递进去；
如果 promise 已经失败，那么执行 onRejected, 并将 promise 失败的原因作为参数传递进去；
如果 promise 的状态是 pending，需要将 onFulfilled 和 onRejected 函数存放起来，等待状态确定后，再依次将对应的函数执行（发布订阅）
7. then 的参数 onFulfilled 和 onRejected 可以缺省
8. promise 可以 then 多次，promise 的 then 方法返回一个 promise
9. 如果 then 返回的是一个结果，那么就会把这个结果作为参数，传递给下一个 then 的成功的回调(onFulfilled)
10. 如果 then 中抛出了异常，那么就会把这个异常作为参数，传递给下一个 then 的失败的回调(onRejected)
11. 如果 then 返回的是一个 promise，那么会等这个 promise 执行完，promise 如果成功，就走下一个 then 的成功，如果失败，就走下一个 then 的失败

## Promise 其他方法

虽然我们已经实现了一个 [Promise/A+](https://promisesaplus.com/) 规范的 Promise，但是相比原生的 Promise 还缺少一些方法:

+ Promise.prototype.catch()
+ Promise.prototype.finally()
+ Promise.resolve()
+ Promise.reject()
+ Promise.all()
+ Promise.race()

下面，我们将逐一实现：

### Promise.prototype.catch

用于指定出错时的回调，是特殊的 then 方法，catch 之后，可以继续 then。

```javascript
Promise.prototype.catch = function (onRejected) {
  return this.then(null, onRejected);
}
```

### Promise.prototype.finally

不管成功还是失败，都会走到 finally 中，并且 finally 之后，还可以继续 then。并且将值原封不动的传递给后面的 then。

```javascript
Promise.prototype.finally = function (callback) {
  return this.then((value) => {
    return Promise.resolve(callback()).then(() => {
      return value;
    });
  }, (err) => {
    return Promise.resolve(callback()).then(() => {
      throw err;
    });
  });
}
```

### Promise.resolve

Promise.resolve(value) 返回一个以给定值解析后的 promise 对象.

1. 如果 value 是个 thenable 对象，返回的 promise 会跟随这个 thenable 的对象，采用它的最终状态
2. 如果传入的 value 本身就是 promise 对象，那么 Promise.resolve 将不做任何修改、原封不动地返回这个 promise 对象
3. 其他情况，直接返回以该值为成功状态的 promise 对象

```javascript
Promise.resolve = function (param) {
  if (param instanceof Promise) {
    return param;
  }
  return new Promise((resolve, reject) => {
    if (param && param.then && typeof param.then === 'function') {
      // 为保持与原生 Promise 对象执行顺序一致，模拟使用 setTimeout
      setTimeout(() => {
        param.then(resolve, reject);
      });
    } else {
      resolve(param);
    }
  });
}
```

### Promise.reject

Promise.reject 方法和 Promise.resolve 不同，Promise.reject() 方法的参数会原封不动地作为 reject 的理由，变成后续方法的参数。

```javascript
Promise.reject = function (reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  });
}
```

### Promise.all

Promise.all(promises) 返回一个 promise 对象。

1. 如果传入的参数是一个空的可迭代对象，那么此 promise 对象回调完成 resolve，只有此情况，是同步执行的，其它都是异步返回的
2. 如果传入的参数不包含任何 promise，则返回一个异步完成
3. 等待 promises 中所有的 promise 执行完毕
4. 如果参数中有一个 promise 失败，那么 Promise.all 返回的 promise 对象失败
5. 在任何情况下，Promise.all 返回的 promise 完成状态的结果都是一个数组

```javascript
Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    let index = 0;
    let result = [];
    if (promises.length === 0) {
      resolve(result);
    } else {
      function processValue(i, data) {
        result[i] = data;
        if (++index === promises.length) {
          resolve(result);
        }
      }
      for (let i = 0; i < promises.length; i++) {
        //promises[i] 可能是普通值
        Promise.resolve(promises[i]).then((data) => {
          processValue(i, data);
        }, (err) => {
          reject(err);
          return;
        });
      }
    }
  });
}
```

### Promise.allSettled

不同于 `Promise.all`，一旦有一个 `promise` 执行失败，就无法获得其他成功 `promise` 返回。`Promise.allSettled` 方法下无论某个 `promise` 成功与否，所有的 `promise` 总是被 `resolve` 的，并且值是所有 `promise` 执行结果的描述。

```javascript
Promise.allSettled = function (promises) {
  return new Promise((resolve, reject) => {
    let index = 0;
    let result = [];
    if (promises.length === 0) {
      resolve(result);
    } else {
      function processValue(i, data) {
        result[i] = data;
        if (++index === promises.length) {
          resolve(result);
        }
      }
      for (let i = 0; i < promises.length; i++) {
        //promises[i] 可能是普通值
        Promise.resolve(promises[i]).then((data) => {
          processValue(i, { status: 'fulfilled', value: data });
        }, (err) => {
          processValue(i, { status: 'rejected', reason: err })
        });
      }
    }
  });
}
```

### Promise.race

顾名思义，Promse.race 就是赛跑的意思。意思就是说，Promise.race([p1, p2, p3]) 里面哪个结果获得的快，就返回那个结果，不管结果本身是成功状态还是失败状态。

1. 如果传的参数数组是空，则返回的 promise 将永远等待
2. 如果迭代包含一个或多个非承诺值和/或已解决/拒绝的承诺，则 Promise.race 将解析为迭代中找到的第一个值

```javascript
Promise.race = function (promises) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      return;
    } else {
      for (let i = 0; i < promises.length; i++) {
        Promise.resolve(promises[i]).then((data) => {
          resolve(data);
          return;
        }, (err) => {
          reject(err);
          return;
        });
      }
    }
  });
}
```

详细代码，同步 [github](https://github.com/fuzhongyi/Promise)
