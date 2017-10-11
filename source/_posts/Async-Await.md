title: ES7 中的 async await
date: 2017-10-11 14:36:46
tags: [async, await]
categories: JS
---

在 JavaScript 中处理异步操作的回调 (callback) 通常会导致多嵌套的代码块，俗称回调地狱 (callback hell)，这样的代码复杂，可读性，可维护性非常不友好；直达 ES6，Promise 出现，使得我们能够扁平化回调函数，告别回调地狱，写出优雅的代码；但是在实践中发现，Promise 并不完美，如果 Promise 的回调中出现嵌套，依旧会出现回调地狱；而如今，async await 出现了，它提供了一种新的编写异步代码方式，使得异步代码看起来像是同步代码，这正式它的魔力所在。

![](http://7vikhl.com1.z0.glb.clouddn.com/1-ko3KtcVSlzpe3RnTRgJaHw.jpeg)

<!-- more -->

## 异步函数
```
// async function statement：async function 声明语句
async function name([param[, param[, ... param]]]) {
   statements
}

// async function expression：async function 表达式
async function [name]([param1[, param2[, ..., paramN]]]) {
   statements
}
```
如上，可以通过 async function 声明语句或者表达式来定义一个异步函数，返回一个 `AsyncFunction` 对象，二者的区别在于在于函数名称，async function 表达式可以省略函数名称来创建一个匿名的函数。

**调用异步函数时会返回一个 promise 对象**
- 当这个异步函数返回一个值时，promise 的 resolve 方法将会处理这个返回值
- 当异步函数抛出异常或者非法值时，promise 的 reject 方法将处理这个异常值

```
async function test () {
  return 'ok'
  // throw new Error('error')
}

test().then(console.log, console.error)
```

## `AsyncFunction`
> 注意，`AsyncFunction` 并不是一个全局对象，我们可以这样获得 `AsyncFunction`
```
Object.getPrototypeOf(async function () {}).constructor
```

`AsyncFunction` 构造函数用于创建一个新的 async function 对象，在 JavaScript 中，每一个异步函数都是一个 `AsyncFunction`
```
console.log(async function () {}.constructor)
// -> ƒ AsyncFunction() { [native code] }

console.log(async function () {}.__proto__)
// -> AsyncFunction {Symbol(Symbol.toStringTag): "AsyncFunction", constructor: ƒ}

var AsyncFunction = Object.getPrototypeOf(async function () {}).constructor
var asyncFun = async function () {}
console.log(asyncFun instanceof AsyncFunction) // -> true
```

通过 `AsyncFunction` 构造函数创建异步函数的语法如下：
```
new AsyncFunction([arg1[, arg2[, ...argN]],] functionBody)
```
把 `AsyncFunction` 当成函数调用（省略 `new` 操作符）也可创建异步函数，如下：

```
var AsyncFunction = Object.getPrototypeOf(async function () {}).constructor;
// var func = new AsyncFunction('a', 'b', 'return a+b')
var func = AsyncFunction('a', 'b', 'return a+b')
func(1, 2).then(console.log) // -> 3
```

👉 通过 `AsyncFunction` 构造函数创建的异步函数 在当前上下文不创建闭包；而是在全局作用域创建的。而且只需效率比用 async function 声明语句或者表达式来定义一个异步函数要低，所以一般情况使用 async function 声明语句或者表达式来定义一个异步函数。

## `await`
`await` 操作符用于等待一个 Promise 返回结果或者某个直接的值，且 `await` 必须在异步函数 (async function) 上下文中使用。
```
function returnPromise (a) {
  return new Promise((resolve, reject) => {
    resolve(a)
  })
}
function returnValue (a) {
  return a
}

async function test () {
// var a = await returnPromise(2)
  var a = await returnValue(3)
  return a
}

test().then(console.log, console.error)
```

异步函数中执行 `await` 表达式，这将会使异步函数暂停执行并等待 promise 解析传值后，继续执行异步函数并返回解析值。
```
function sleep (second) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, second * 1000)
  })
}

async function test () {
  console.log(new Date())
  await sleep(3)
  console.log(new Date())
}

test()
```
代码执行结果：
```
Wed Oct 11 2017 16:00:32 GMT+0800 (CST)
Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
Wed Oct 11 2017 16:00:35 GMT+0800 (CST)
```

`await` 串行，并行执行
```
function multi (a) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve.bind(null, a * a), 1000)
  })
}

// 串行执行
async function test () {
   var x = await multi(2)
   var y = await multi(3)
   return [x, y]
}

// 并行执行
// async function test () {
//    var res = await Promise.all([multi(2), multi(3)])
//    return res
// }

test().then(console.log)
```

## async await 的优势
Promise 有了 async await 的辅助，使其发挥更大的威力，通过 async await 终于可以彻底摆脱回调地狱，以同步方式编写异步代码，代码简洁，十分友好，下面举几个例子 👇

### 错误处理
`try-catch` 处理同步，异步错误。如下代码片段一中，`try-catch` 并不能捕捉 `then` 方法中 `JSON.parse` 异常出错；但是在代码片段二中，使用了 async await，使得 `try-catch` 能捕捉 `JSON.parse` 可能的异常出错。
```
// 代码块一
function test () {
  try {
    getJSON().then((res) => {
      var data = JSON.parse(res) // 此处可能出错
    })
  } catch (e) {
    // ...
  }
}

// 代码块二
async function test () {
  try {
    var data = JSON.parse(await getJSON())
  } catch (e) {
    // ...
  }
}
```

### 中间值
有时会遇到这样的情形：promise1 返回值 value1；promise2 依赖 value1，返回value2；promise3 依赖 value1 和 value2。最简单的做法是通过嵌套解决 promise 间的依赖：
```
function test () {
  return promise1()
      .then(value1 => {
        return promise2(valu1)
          .then(value2 => {
              return promise3(value1, value2)
          })
      })
}
```

显然，这种简单粗暴的处理方式使我们又掉进回调地狱了；我们可以通过中间变量来抹平这个回调嵌套：
```
function test () {
  var value1
  var value2
  return promise1()
    .then(vle => {
        value1 = vle
        return promise2(value1)
    })
    .then(vle => {
        value2 = vle
        return promise3(value1, value2)
    })
}
```

而通过 async await，可以最优，最简洁的解决这个问题：
```
async function test () {
  var value1 = await promise1()
  var value2 = await promise2(value1)
  return promise3(value1, value2)
}
```

❗️注意，在上面代码的返回语句 promise3 并没有 `await`，因为异步函数的会将其返回值隐式封装在 `Promise.resolve` 中。

## 参考链接
[MDN-AsyncFunction](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction)
[MDN-async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
[MDN-await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
