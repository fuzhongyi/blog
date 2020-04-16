---
title: 一文看尽 redux，中间件设计实现
date: 2020-04-15 17:30:57
tags: [javascript,react]
categories : 技术水波文
---


## redux

`React` 作为一个组件化开发框架，组件之间存在大量通信，有时这些通信跨越多个组件，或者多个组件之间共享一套数据，简单的父子组件间传值不能满足我们的需求，自然而然地，我们需要有一个地方存取和操作这些公共状态。而 `redux` 就为我们提供了这样一种管理公共状态的方案。

了解了什么是 `redux`，以及它的到来为我们解决了什么问题，我们围绕这一功能，逐步实现。

既然是公共状态，那么我们创建这样一个文件 `store.js`，然后直接在里边存放公共的 state，其他组件只要引入这个 store 就可以存取共用状态了。

```js
export const state = {
    count: 0
}
```

当然我们并不能这样去设计，主要原因有 2 点：

+ 容易误操作 - 其他任何地方均可以修改 state，这样做显然不太安全 出错了也很难排查，因此我们需要有条件地操作 store，防止使用者直接修改 store 的数据
+ 可读性很差 - js 是一门极其依赖语义化的语言，试想如果在代码中不经注释直接修改了公用的 state，难以维护，为了搞清楚修改 state 的含义还得根据上下文推断，所以我们最好是给每个操作起个名字

思考：

我们希望公共状态既能够被全局访问到，又是私有的不能被直接修改，闭包完全满足这两条要求，因此我们会把公共状态设计成闭包。

既然我们要存取状态，那么肯定要有 `getter` 和 `setter`，此外当状态发生改变时，我们得进行广播，通知组件状态发生了变更。

这不就和 `redux` 的三个 api：`getState`、`dispatch`、`subscribe` 对应上了吗。

我们用几句代码勾勒出 store 的大致形状：

```js
export const createStore = () => {
    let currentState = {}   // 公共状态
    function getState() {}  // getter
    function dispatch() {}  // setter
    function subscribe() {} // 发布订阅
    return { getState, dispatch, subscribe }
}
```

### getState 实现

`getState()` 的实现非常简单，返回当前状态即可：

```js
export const createStore = () => {
    let currentState = {}   // 公共状态
    function getState() {   // getter
        return currentState;
    }  
    function dispatch() {}  // setter
    function subscribe() {} // 发布订阅
    return { getState, dispatch, subscribe }
}
```

### dispatch 实现

经过上面的分析，我们的目标是有条件地、具名地修改 store 的数据。所以这里我们像 redux 中 dispatch 那样传入一个 action 对象，这个对象包含操作类型 type，根据 type 的不同对应修改返回不同的 state。

```js
export const createStore = () => {
    let currentState = {}
    function getState() {
        return currentState
    }
    function dispatch(action) {
        switch (action.type) {
            case 'plus':
            currentState = {
                ...state,
                count: currentState.count + 1
            }
        }
    }
    function subscribe() {}
    return { getState, subscribe, dispatch }
}
```

但是这样我们把执行逻辑写在了 dispatch 中，耦合度太高，于是我们想到把这部分代码抽离出来放到外面。

```js
export const createStore = (reducer) => {
    let currentState = {}
    function getState() {
        return currentState
    }
    function dispatch(action) {
       currentState = reducer(currentState, action)
    }
    function subscribe() {}
    return { getState, subscribe, dispatch }
}
```

这不就是我们熟悉的 `reducer` 吗，然后我们创建一个 reducer.js 文件，写我们的 reducer。

```js
const initialState = {
    count: 0
}
export function reducer(state = initialState, action) {
    switch(action.type) {
        case 'plus':
        return {
            ...state,
            count: state.count + 1
        }
        case 'subtract':
        return {
            ...state,
            count: state.count - 1
        }
        default:
        return initialState
    }
}
```

代码写到这里，我们可以验证一下 `getState` 和 `dispatch`。

```js
import { reducer } from './reducer'

const store = createStore(reducer)  // 创建store
store.dispatch({ type: 'plus' })    // 执行加法操作，给 count 加 1
console.log(store.getState())       // 获取 state
```

运行代码，我们会发现，打印得到的 state 是：`{ count: NaN }`。这是因为，store 中 state 的初始数据为 {}， `state.count + 1` 实际上是 `underfind + 1`。

所以我们得先进行 state 数据的初始化：

```js
export const createStore = (reducer) => {
    let currentState = {}
    function getState() {
        return currentState
    }
    function dispatch(action) {
       currentState = reducer(currentState, action)
    }
    dispatch({type:'@STATE_INIT@'}); // 初始化 state 数据
    function subscribe() {}
    return { getState, subscribe, dispatch }
}
```

这样我们就能得到正确的 state：`{ count: 1 }`。

### subscribe 实现

尽管我们已经能够存取公用 state，但 store 的变化并不会直接引起视图的更新，我们需要监听 store 的变化。每次 dispatch，都进行广播，通知组件 store 的状态发生了变更，这里我们应用一个设计模式——观察者模式。

```js
export const createStore = (reducer) => {
    let currentState = {}
    let observers = []          // 观察者队列
    function getState() {
        return currentState
    }
    function dispatch(action) {
       currentState = reducer(currentState, action)
       observers.forEach(fn => fn())
    }
    dispatch({type:'@STATE_INIT@'}); // 初始化 state 数据
    function subscribe(fn) {
        observers.push(fn)
    }
    return { getState, subscribe, dispatch }
}

const store = createStore(reducer)  
store.subscribe(() => { console.log('组件 1 收到 store 的通知') })
store.subscribe(() => { console.log('组件 2 收到 store 的通知') })
store.dispatch({ type: 'plus' }) // 执行 dispatch，触发 store 的通知
```

到这里，一个简单的 redux 就已经完成。

但我们在使用 store 时，需要在每个组件中 `import` 引入 store，然后获取状态`(getState)`，修改状态`(dispatch)` ，订阅更新`(subscribe)`，代码比较冗余，我们需要合并一些重复操作，而其中一种简化合并的方案，就是我们熟悉的 `react-redux`。

## react-redux

上文我们说到，一个组件如果想从 store 存取公用状态，需要进行四步操作。

react-redux 提供 `Provider` 和 `connect` 两个 API，`Provider` 将 `store` 放进 `this.context` 里，省去了 `import` 这一步，`connect` 将 `getState`、`dispatch` 合并进了 `this.props`，并自动订阅更新，简化了另外三步，下面我们来看一下如何实现这两个 API：

### Provider 实现

Provider 是一个组件，接收 store 并将其放进全局 context。

```js
import React, { Component } from 'react'
import PropTypes from 'prop-types'

export class Provider extends Component {  
    // 需要声明静态属性 childContextTypes 来指定 context 对象的属性，是 context 的固定写法  
    static childContextTypes = {
        store: PropTypes.object  
    }

    // 实现 getChildContext 方法，返回 context 对象，也是固定写法  
    getChildContext() {
        return { store: this.store }  
    }  

    constructor(props, context) {
        super(props, context)
        this.store = props.store  
    }  

    // 渲染被 Provider 包裹的组件  
    render() {
        return this.props.children  
    }
}
```

完成 Provider 后，我们就不需要再单独 import store，可直接在组件中通过 `this.context.store` 取到 store。

### connect 实现

下面我们来思考一下如何实现 `connect`，我们先回顾一下 `connect` 的使用方法：

```js
connect(mapStateToProps, mapDispatchToProps)(App)
```

我们知道，`connect` 接收 `mapStateToProps`、`mapDispatchToProps` 两个方法，然后返回一个高阶函数，这个高阶函数接收一个组件，返回一个高阶组件，其实目的就是给传入组件增加一些属性和功能，connect 根据传入的 `mapStateToProps`、`mapDispatchToProps`，将 state 和 dispatch(action) 挂载到子组件的 props 上：

```js
import React from 'react'

export function connect(mapStateToProps, mapDispatchToProps) {
    return function(Component) {
        class Connect extends React.Component {
            componentDidMount() {
                // 从 context 获取 store 并订阅更新
                this.context.store.subscribe(this.handleStoreChange.bind(this));
            }
            handleStoreChange() {
                // 触发更新
                // 触发的方法有多种，这里为了简洁起见，直接 forceUpdate 强制更新，读者也可以通过 setState 来触发子组件更新
                this.forceUpdate()
            }
            render() {
                return (
                    <Component
                        // 传入该组件的 props，需要由 connect 这个高阶组件原样传回原组件
                        { ...this.props }
                        // 根据 mapStateToProps 把 state 挂到 this.props 上
                        { ...mapStateToProps(this.context.store.getState()) }
                        // 根据 mapDispatchToProps 把 dispatch(action) 挂到 this.props 上
                        { ...mapDispatchToProps(this.context.store.dispatch) }
                    />
                )
            }
        }
        // 接收 context 的固定写法
        Connect.contextTypes = {
            store: PropTypes.object
        }
        return Connect
    }
}
```

## redux Middleware

所谓中间件，我们可以理解为拦截器，用于对某些过程进行拦截和处理，且中间件之间能够串联使用。在 redux 中，我们中间件拦截的是 dispatch 提交到 reducer 这个过程，从而增强 dispatch 的功能。

可查阅[官方文档](https://redux.js.org/advanced/middleware/)。

接下来让我们和官方文档一样，以一个记录日志的中间件为例，一步一步分析 redux 中间件的设计实现。

### 在每次 dispatch 之后手动打印 store 的内容

假如我们想在每次 dispatch 之后，打印一下 store 的内容，我们则会这样去实现：

```js
store.dispatch({ type: 'plus' })
console.log('next state', store.getState())
```

这是最简单直接的实现方式，当然我们并不可能在项目每个 dispatch 的地方都添加这样一段代码，我们可以把这部分功能的代码提取出来。

### 封装 dispatch

```js
function patchStoreToAddLogging(store, action) {
    store.dispatch(action)
    console.log('next state', store.getState())
}
```

这样可以减少一部分重复的代码。不过每次使用这个新的 dispatch 都得从外部引一下，依然没有解决增加不必要的业务代码。

所以我们得思考另外一种一劳永逸的方式，替换 dispatch：

### 替换 dispatch

```js
const next = store.dispatch
store.dispatch = function patchStoreToAddLogging(action) {  
    let result = next(action)  
    console.log('next state', store.getState())  
    return result
}
```

这样我们每次使用的时候就不需要再从外部引用一次了。

但这样还是不够好的，试想一下，如果我们又有其他的需求，比方监控 dispatch 错误，我们固然可以在打印日志的代码后面加上捕获错误的代码，但随着功能模块的增多，代码量也会迅速膨胀，导致难以维护。

所以，我们希望不同的功能是独立的可拔插的模块。

### 模块化

```js
// 打印 state 中间件
function patchStoreToAddLogging(store) {
    const next = store.dispatch    //此处也可以写成匿名函数
    store.dispatch = function(action) {
        let result = next(action)
        console.log('next state', store.getState())
        return result
    }
}  

// 监控错误中间件
function patchStoreToAddCrashReporting(store) {
    // 这里取到的 dispatch 已经是被上一个中间件包装过的 dispatch，从而实现中间件串联
    const next = store.dispatch
    store.dispatch = function(action) {
        try {
            return next(action)
        } catch (err) {
            console.error('捕获一个异常!', err)
            throw err
        }
    }
}

patchStoreToAddLogging(store)
patchStoreToAddCrashReporting(store)
```

到这里我们基本实现了可组合、拔插的中间件。

### applyMiddleware

我们注意到，我们当前写的中间件方法都是先获取 dispatch，然后在方法内替换 dispatch，这部分重复代码我们可以再稍微简化一下。

我们不在方法内替换 dispatch，而是返回一个新的 dispatch，然后让循环来进行每一步的替换：

```js
import { reducer } from './reducer'

function logger(store) {
    const next = store.dispatch
    return function(action) {
        let result = next(action)
        console.log('next state', store.getState())
        return result
    }
}

function crashReporter(store) {
    const next = store.dispatch
    return function(action) {
        try {
            return next(action)
        } catch (err) {
            console.error('捕获一个异常!', err)
            throw err
        }
    }
}

function applyMiddleware(store, middlewares) {
    middlewares = middlewares.slice()   // 返回新数组，避免 reserve() 影响原数组
    middlewares.reverse()               // 由于循环替换 dispatch 时，前面的中间件在最里层，因此需要翻转数组才能保证中间件的调用顺序
    // 循环替换 dispatch
    middlewares.forEach(middleware => (store.dispatch = middleware(store)))
}

applyMiddleware(store, [ logger, crashReporter ])
```

### 存函数

之前的例子已经基本实现我们的需求，但我们还可以进一步改进，上面这个函数看起来仍然不够”纯“，函数在函数体内修改了 store 自身的 dispatch ，产生了所谓的“副作用”，从函数式编程的规范出发，我们可以进行一些改造，借鉴 react-redux 的实现思路，我们可以把 applyMiddleware 作为高阶函数，用于增强 store，而不是替换 dispatch。

```js

const logger = store => next => action => {
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('捕获一个异常!', err)
    throw err
  }
}

const thunk = store => next => action =>
  typeof action === 'function'
    ? action(store.dispatch, store.getState)
    : next(action)

const applyMiddleware = (...middlewares) => createStore => reducer => {
    const store = createStore(reducer)
    let { getState, dispatch } = store
    const params = {
        getState,
        // 如果所有中间件都共享同一个 dispatch，出现中间件修改了 dispatch 或者进行异步 dispatch 就可能出错
        dispatch: (action) => dispatch(action)
    }
    const middlewareArr = middlewares.map(middleware => middleware(params))
    dispatch = compose(...middlewareArr)(dispatch)
    return { ...store, dispatch }
}

// compose 这一步对应了 middlewares.reverse()，是函数式编程一种常见的组合方法
// 使传入的中间件函数变成 (...arg) => mid1(mid2(mid3(...arg)))
function compose(...funcs) {
    if (funcs.length === 0) return arg => arg
    if (funcs.length === 1) return funcs[0]
    return funcs.reduce((a, b) =>(...args) => a(b(...args)))
}

export const createStore = (reducer, enhancer) => {
    if(applyMiddleware) {
       return enhancer(createStore)(reducer)
    }
    let currentState = {}
    let observers = []          // 观察者队列
    function getState() {
        return currentState
    }
    function dispatch(action) {
       currentState = reducer(currentState, action)
       observers.forEach(fn => fn())
    }
    dispatch({type:'@STATE_INIT@'}); // 初始化 state 数据
    function subscribe(fn) {
        observers.push(fn)
    }
    return { getState, subscribe, dispatch }
}

createStore(reducer, applyMiddleware(logger, crashReporter, thunk))
```
