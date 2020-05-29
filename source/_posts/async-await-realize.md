---
title: async/await 实现原理
date: 2020-05-29 17:56:43
tags: [javascript, async]
---

## 使用

```js
var p1 = () => new Promise(resolve => setTimeout(() => resolve('Promisem 1'), 3000));
var p2 = () => new Promise(resolve => setTimeout(() => resolve('Promisem 2'), 3000));

async function test() {
  var data1 = await p1();
  console.log(data1);
  var data2 = await p2();
  console.log(data2);
  return 'end';
}

test().then(console.log);
```

## 实现

```js
var p1 = () => new Promise(resolve => setTimeout(() => resolve('Promisem 1'), 3000));
var p2 = () => new Promise(resolve => setTimeout(() => resolve('Promisem 2'), 3000));

function* test() {
  var data1 = yield p1();
  console.log(data1);
  var data2 = yield p2();
  console.log(data2);
  return 'end';
}

function asyncToGenerator(gen) {
  return new Promise((resolve, reject) => {
    var g = gen.apply(this, Array.prototype.slice.call(arguments, 1));
    function step(val) {
      var result = g.next(val);
      if (result.done) {
        resolve(result.value);
      } else {
        Promise.resolve(result.value).then(step, reject);
      }
    }
    step();
  });
}

asyncToGenerator(test).then(console.log);
```
