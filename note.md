# 阅读Redux


## Redux 的适用场景 ==> 多交互、多数据源

从业务角度看，如果你的应用有以下场景，可以考虑使用 Redux。

- 用户的使用方式复杂
- 不同身份的用户有不同的使用方式（比如普通用户和管理员）
- 多个用户之间可以协作
- 与服务器大量交互，或者使用了WebSocket
- View要从多个来源获取数据

从组件角度看，如果你的应用有以下场景，可以考虑使用 Redux。

- 某个组件的状态，需要共享
- 某个状态需要在任何地方都可以拿到
- 一个组件需要改变全局状态
- 一个组件需要改变另一个组件的状态

## 设计思想

Redux 的设计思想很简单，就两句话。

（1）Web 应用是一个状态机，视图与状态是一一对应的。

（2）所有的状态，保存在一个对象里面。

## 三大原则

1. 单一数据源

整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。

2. State 是只读的

唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。

3. 使用纯函数来执行修改

为了描述 action 如何改变 state tree ，你需要编写 reducers。


## API

1. createStore

它可以接受三个参数，reducer、preloadedState、enhancer：

- reducer：是一个函数，返回下一个状态，接受两个参数：当前状态 和 触发的 action；
- preloadedState：初始状态对象，可以很随意指定，比如服务端渲染的初始状态，但是如果使用 combineReducers 来生成 reducer，那必须保持状态对象的 key 和 combineReducers 中的 key 相对应；
- enhancer：store 的增强器函数，可以指定为 第三方的中间件，时间旅行，持久化 等等，但是这个函数只能用 Redux 提供的 applyMiddleware 函数来生成；

2. combineReducers
3. bindActionCreators,
4. applyMiddleware

```js
(createStore) => (reducer, preloadedState, enhancer) => {
  // ...
}
```
5. compose

```ts
function compose(...funcs: Function[]) {
  if (funcs.length === 0) {
    // infer the argument type so it is usable in inference down the line
    return <T>(arg: T) => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce(
    (a, b) =>
      (...args: any) =>
        a(b(...args))
  )
}

// 使用reduce实现的
```

## 有用的代码段

```js
// Inlined / shortened version of `kindOf` from https://github.com/jonschlinkert/kind-of
export function miniKindOf(val: any): string {
  if (val === void 0) return 'undefined'
  if (val === null) return 'null'

  const type = typeof val
  switch (type) {
    case 'boolean':
    case 'string':
    case 'number':
    case 'symbol':
    case 'function': {
      return type
    }
  }

  if (isArray(val)) return 'array'
  if (isBuffer(val)) return 'buffer'
  if (isDate(val)) return 'date'
  if (isError(val)) return 'error'

  const constructorName = ctorName(val)
  switch (constructorName) {
    case 'Symbol':
    case 'Promise':
    case 'WeakMap':
    case 'WeakSet':
    case 'Map':
    case 'Set':
      return constructorName
  }

  // other
  return Object.prototype.toString
    .call(val)
    .slice(8, -1)
    .toLowerCase()
    .replace(/\s/g, '')
}

function ctorName(val: any): string | null {
  return typeof val.constructor === 'function' ? val.constructor.name : null
}

function isError(val: any) {
  return (
    val instanceof Error ||
    (typeof val.message === 'string' &&
      val.constructor &&
      typeof val.constructor.stackTraceLimit === 'number')
  )
}

function isDate(val: any) {
  if (val instanceof Date) return true
  return (
    typeof val.toDateString === 'function' &&
    typeof val.getDate === 'function' &&
    typeof val.setDate === 'function'
  )
}

function isArray(val: any) {
  if (Array.isArray) return Array.isArray(val)
  return val instanceof Array
}

/**
 * If you need to support Safari 5-7 (8-10 yr-old browser),
 * take a look at https://github.com/feross/is-buffer
 */

 function isBuffer(val: any) {
  if (val.constructor && typeof val.constructor.isBuffer === 'function') {
    return val.constructor.isBuffer(val);
  }
  return false;
}

function isGeneratorObj(val: any) {
  return typeof val.throw === 'function'
    && typeof val.return === 'function'
    && typeof val.next === 'function';
}

/**
 * @param obj The object to inspect.
 * @returns True if the argument appears to be a plain object.
 */
function isPlainObject(obj: any): boolean {
  if (typeof obj !== 'object' || obj === null) return false

  let proto = obj
  while (Object.getPrototypeOf(proto) !== null) {
    proto = Object.getPrototypeOf(proto)
  }

  return Object.getPrototypeOf(obj) === proto
}

```

> 参考：https://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html