title: Implementing Promise
date: 2017-09-06 22:10:50
tags: Promise
categories: JS
---

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/PromisesA%2B.png)

<!-- more -->

本文将剖析 Promise 的内部结构，实现符合 [Promises/A+](https://promisesaplus.com/) 规范的 Promise，并且通过 [promises-aplus/promises-tests](https://github.com/promises-aplus/promises-tests#promisesa-compliance-test-suite) 测试。

## JavaScript Promise 标准解读
Promise 表示一个异步操作的最终结果，与之进行交互的方式主要是 `then` 方法，该方法注册了两个回调函数，用于接收 Promise resolve 的终值或者 Promise reject 的原因。
>详细 **Promise A+ 规范** 参考：[英文](https://promisesaplus.com/)  [中文](https://malcolmyu.github.io/2015/06/12/Promises-A-Plus/)

### Promise States
- 一个 Promise 的当前状态必须为以下三种状态中的一种:
	- PENDING
	- FULFILLED
	- REJECTED
- Promise 的初始状态为 PENDING，由此状态可转换为 FULFILLED 或者 REJECTED，一旦状态确定，就不能再次转换为其他状态，此状态称之为 settle

### `then` 方法
- 一个 Promise 必须提供 `then` 方法，该方法接收两个参数：
```
promise.then(onFulfilled, onRejected)
```
- `then` 方法必须返回一个 Promise 对象
```
promise2 = promise1.then(onFulfilled, onRejected)
```

### The Promise Resolution Procedure
不同实现的 Promise 需要可以无缝相互调用(interoperable)，例如：
```
// MyPromise 表示自己实现的 Promise
new MyPromise((resolve, reject) => {
    resolve(1)
}).then(() => {
    return Promise.reject(2) // ES6 Promise
})
```

## 实现 Promise
### Promise 构造函数
```
class Promise {
  // The constructor function
  constructor (executor) {
    if (!this._isFunction(executor)) {
      throw new TypeError('Promise argument error:' + executor.toString())
    }

    this._status = PENDING
    this._v = undefined
    this._onResolvedCb = []
    this._onRejectedCb = []

    executor(this.resolve.bind(this), this.reject.bind(this))
  }
}

var promise = new Promise(function(resolve, reject) {
  // 执行操作...
  // 操作成功：调用 resolve，传入 value
  resolve(value)
  // 操作失败: 调用 reject，传入 reason
  reject(reason)
})
```

- 构造函数中，`_onResolvedCb` 和 `_onRejectedCb` 属性分别为 Promise resolved/rejected 的回调函数集，因为在 Promise 结束（settle）之前，可能有多个 `then` 挂在该 promise 上，即有多个回调事件；
- 构造函数接收一个 executor 函数，函数中的操作执行结束后，若成功，调用 resolve 并传入 value；若失败，调用 reject 并传入 reason

接下来实现 `resolve` 和 `reject` 这两个函数，这两个函数类似，首先判断当前 Promise 状态后，变更其状态；并记录 resolved 的终值和 rejected 的原因；最后执行回调函数
```
resolve (value) {
  if (this._status !== PENDING) {
    return
  }

  this._status = FULFILLED
  this._v = value

  let fn
  while ((fn = this._onResolvedCb.shift())) {
    fn.call(this, value)
  }
}

reject (reason) {
  // ...
}
```

### `then` 方法
Promise 对象的 `then` 方法用来注册 Promise resolved 或者 rejected 后的回调函数，`then` 方法返回一个新的 Promise 对象
```
then (onResolved, onRejected) {
  return new Promise((resolve, reject) => {
    switch (this._status) {
      case PENDING:
        // ...
        break
      case FULFILLED:
        // ...
        break
      case REJECTED:
        // ...
        break
    }
  })
}
```
- `then` 方法返回的新 Promise 对象中，需要根据前一个 Promise 对象的不同状态来处理：
  - 若是 FULFILLED，则直接执行 `onResolved` 回调函数。如果 `onResolved` 的返回值又是一个 Promise 对象，那么继续通过 `then` 方法构造 Promise 链；否则，调用 `resolve` 函数结束新的 Promise 对象
  - 若是 REJECTED，处理逻辑同 FULFILLED
  - 若是 PENDING，则将 `onResolved` 和 `onRejected` 回调函数放到回调函数集中

最终得到如下简易版本的 `then` 方法：
```
then (onResolved, onRejected) {
  // Promise 值穿透
  onResolved = this._isFunction(onResolved) ? onResolved : function (v) { return v }
  onRejected = this._isFunction(onRejected) ? onRejected : function (r) { throw r }

  var promise2 = new Promise((resolve, reject) => {
    switch (this._status) {
      case PENDING:
        this._onResolvedCb.push((value) => {
          // 执行 onResolved 或者 onRejected 可能出现异常，需要 try-catch
          try {
            var x = onResolved(value)
            if (x instanceof Promise) {
              x.then(resolve, reject)
            } else {
              resolve(x)
            }
          } catch (e) {
            reject(e)
          }
        })
        this._onRejectedCb.push((reason) => {
          try {
            var x = onRejected(reason)
            if (x instanceof Promise) {
              x.then(resolve, reject)
            } else {
              resolve(x)
            }
          } catch (e) {
            reject(e)
          }
        })
        break
      case FULFILLED:
        setTimeout(() => {
          try {
            var x = onResolved(this._v)
            if (x instanceof Promise) {
              x.then(resolve, reject)
            } else {
              resolve(x)
            }
          } catch (e) {
            reject(e)
          }
        })
        break
      case REJECTED:
        setTimeout(() => {
          try {
            var x = onRejected(this._v)
            if (x instanceof Promise) {
              x.then(resolve, reject)
            } else {
              resolve(x)
            }
          } catch (e) {
            reject(e)
          }
        })
        break
    }
  })
  return promise2
}
```

**代码剖析注意：**
1、`then` 方法的两个回调函数执行过程中可能抛出异常，所以在执行时，需要 `try-catch` 包起来
2、Promise 值穿透
```
new Promise((resolve, reject) => {
  resolve(1)
})
.then()
.then((value) => {
  console.log('resolve: ', value)
}, (reason) => {
  console.log('reject: ', reason)
})
```
如上代码正确的执行结果为输出：1；注意到第一个 `then` 方法并没有传入回调函数，所以在执行的时候，会抛出异常，提示：`TypeError: onResolved is not a function`；在添加 `try-catch` 后，可以正常执行；catch 到异常后，执行 `reject(e)`，于是第二个 `then` 方法中的 reject 回调函数执行了。

为了解决这个问题，我们需要判断 `then` 方法传入的两个参数是否为函数，若不是，直接返回 value 或者抛出 reason。
```
onResolved = this._isFunction(onResolved) ? onResolved : function (v) { return v }
onRejected = this._isFunction(onRejected) ? onRejected : function (r) { throw r }
```

### 不同 Promise 的交互
`then` 方法传入的两个回调函数 `onResolved` 和 `onRejected` 执行后，返回值 `x` 可能是一个 Promise 对象，也即是 `thenable`，为了确保调用 `x` 上的 `then` 方法成功，我们需要实现标准中 **[2.3.The Promise Resolution Procedure](https://promisesaplus.com/#the-promise-resolution-procedure)** 的内容，这样即使实现方式不同，但遵循标准，不同实现的 Promise 之间就可以交互使用了。

标准对此说明的非常详细，对照着标准阅读以下代码，有助理解。
```
// The Promise Resolution Procedure
// https://promisesaplus.com/#the-promise-resolution-procedure
static resolvePromise (promise2, x, resolve, reject) {
  var then
  // multiple calls to the same argument are made, the first call takes precedence, and any further calls are ignored
  var hasBeenCalled = false

  // 2.3.1 If promise and x refer to the same object
  if (promise2 === x) {
    reject(new TypeError('Chaining cycle detected for promise!'))
    return
  }

  // 2.3.2 If x is a promise
  if (x instanceof Promise) {
    if (x._status === PENDING) { // 2.3.2.1
      x.then(function (v) {
        Promise.resolvePromise(promise2, v, resolve, reject)
      }, reject)
    } else { // 2.3.2.2  2.3.2.3
      x.then(resolve, reject)
    }
    return
  }

  // 2.3.3 Otherwise, if x is an object or function
  if ((x !== null) && ((typeof x === 'object') || (typeof x === 'function'))) {
    try {
      then = x.then // 2.3.3.1
      if (typeof then === 'function') { // 2.3.3.3
        then.call(x, function rs (y) { // 2.3.3.3.1
          if (hasBeenCalled) return // 2.3.3.3.3
          hasBeenCalled = true
          Promise.resolvePromise(promise2, y, resolve, reject)
        }, function rj (r) { // 2.3.3.3.2
          if (hasBeenCalled) return // 2.3.3.3.3
          hasBeenCalled = true
          reject(r)
        })
      } else { // 2.3.3.4
        resolve(x)
      }
    } catch (e) { // 2.3.3.2
      if (hasBeenCalled) return // 2.3.3.3.4.1
      hasBeenCalled = true
      reject(e) // 2.3.3.3.4.2
    }
  } else { // 2.3.4
    resolve(x)
  }
}
```
在 `then` 方法中需要修改，`onResolved` 和 `onRejected` 执行后，调用如上的 `resolvePromise (promise2, x, resolve, reject)` 方法
```
...
case FULFILLED:
  setTimeout(() => {
    try {
      var x = onResolved(this._v)
      Promise.resolvePromise(promise2, x, resolve, reject)
    } catch (e) {
      reject(e)
    }
  })
  break
case REJECTED:
  setTimeout(() => {
    try {
      var x = onRejected(this._v)
      Promise.resolvePromise(promise2, x, resolve, reject)
    } catch (e) {
      reject(e)
    }
  })
  break
...
```

## Promises/A+ test
Promises/A+ 规范提供了一个测试规范： [promises-aplus/promises-tests](https://github.com/promises-aplus/promises-tests#promisesa-compliance-test-suite)
测试需要提供一个 adapter 方法 `Promise.deferred`
```
'use strict'

import Promise from './promise.js'

Promise.deferred = function () {
  var dfd = {}
  dfd.promise = new Promise(function (resolve, reject) {
    dfd.resolve = resolve
    dfd.reject = reject
  })
  return dfd
}
export default Promise
```

执行：
```
promises-aplus-tests dist/promise-adapter.js
```

## 代码
- 简易版(没有实现 Promises/A+ 标准 2.3.The Promise Resolution Procedure)
[YingshanDeng / simple-promise.js](https://gist.github.com/YingshanDeng/21ef2abab6cede0ae2bcd00986f762fd)
- 完整版(完全通过 Promises/A+ 测试规范)
[YingshanDeng/Promise](https://github.com/YingshanDeng/Promise)

## 参考链接
[剖析Promise内部结构，一步一步实现一个完整的、能通过所有Test case的Promise类](https://github.com/xieranmaya/blog/issues/3)
