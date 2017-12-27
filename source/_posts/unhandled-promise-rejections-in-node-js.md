title: Node.js 中的 UnhandledPromiseRejectionWarning 问题
date: 2017-12-27 17:25:07
tags: Promise
categories: JS
---

*问题引入：*今天在 Gulp 构建任务中出现一个 html 解析错误，但是并没有报错，也没有中断 gulp 构建任务的执行，而是出现 `UnhandledPromiseRejectionWarning` 的警告，所以会误以为构建成功，这篇文章将对此进行探究并解决该问题。

```
(node:24866) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 2): Error in plugin 'gulp-posthtml'
Message:
  Parse Error: <img id="titleIcon" class$="{{getStypeType_(info.stype)}}" src$="{{getTitleIcon_(in
  ...
(node:24866) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```
<!-- more -->

## 关于 Unhandled Rejection
一个 Promise 是一个异步操作的状态机，其可能处于这三种状态之一
- `pending`：异步操作还在执行中
- `fulfilled`：异步操作已经完成
- `rejected`：异步操作执行失败

> Node.js 6.6.0 added a sporadically useful bug/feature: **logging unhandled promise rejections to the console by default.**

在 Node.js 6.6.0 中增加了一个特性：对 Promise 中未处理的 rejection 默认会输出 `UnhandledPromiseRejectionWarning` 提示

例如：`test.js` 中有如下代码：
```
new Promise((resolve, reject) => {
  setTimeout(() => reject('woops'), 500);
});
```

`node test.js` 执行：
```
(node:47122) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): error
(node:47122) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code
```

另一种情况是直接在 Promise 中抛出异常：
```
new Promise(() => { throw new Error('exception!'); });
```

执行后也会有 `UnhandledPromiseRejectionWarning` 的警告：
```
(node:47657) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: exception!
(node:47657) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```

Promise API 中有 `.catch()` 这个方法，可以用来处理捕捉 rejection 进行处理
```
new Promise((resolve, reject) => {
  setTimeout(() => reject('error'), 500);
})
.catch(error => console.log('caught', error))


new Promise(() => { throw new Error('exception!'); })
.catch(error => console.log('caught', error.message))
```

但是注意：
```
new Promise((_, reject) => reject(new Error('woops')))
.catch(error => { console.log('caught', err.message); });
```
这个例子中虽然用 `.catch()` 捕捉处理了 Promise 中的 rejection；但是注意在 `err.message` 中的 `err` 是未定义的，代码执行时会抛出错误，由于没有后续的处理，所以也会输出 `UnhandledPromiseRejectionWarning` 的警告

```
(node:47918) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 2): ReferenceError: err is not defined
(node:47918) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```

所以稍不注意就会引起 Promise 中的 **unhandled rejections** 😨

## `unhandledRejection` 事件
在 node `process` 中有一个 `unhandledRejection` 事件，当没有对 Promise 的 rejection 进行处理就会抛出这个事件（这只对原生 Promise 有效）
> The **unhandledrejection** event is fired when a JavaScript Promise is rejected but there is no rejection handler to deal with the rejection.

```
process.on('unhandledRejection', error => {
  // Will print "unhandledRejection err is not defined"
  console.log('unhandledRejection', error.message);
});

new Promise((_, reject) => reject(new Error('woops')))
.catch(error => { console.log('caught', err.message); });
```

此时执行后，就没有 `UnhandledPromiseRejectionWarning` 的警告输出了，只输出：`unhandledRejection err is not defined`

如果我们不想监听 `unhandledRejection` 事件，也不想看到 `UnhandledPromiseRejectionWarning` 的警告输出，怎么办呢？
```
new Promise((_, reject) => reject(new Error('woops')))
.catch(new Function());
```

我们可以在 `.catch()` 中传入一个空函数，假装对 rejection 进行了处理，这样也没有触发 `unhandledRejection` 事件

## Async/Await
关于 Async/Await，可以参考文章：[ES7 中的 async await](https://objcer.com/2017/10/11/Async-Await/)，在这篇文章中详细介绍了 Async/Await 并且和 Promise 进行了对比，Async/Await 在处理异步操作上的优势更明显。

```
async function test() {
  // No unhandled rejection!
  await Promise.reject(new Error('test'));
}

test();
// 输出：
// (node:54358) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 3): Error: test
// (node:54358) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.

test().catch(error => console.log(error.message));
// 输出：
// test
```

async 异步函数返回的是 Promise，所以执行异步函数后，统一需要用 `.catch()` 对可能出现的 rejection 进行捕捉处理，否则统一也是会出现 `UnhandledPromiseRejectionWarning` 的警告

## 解决问题
最后解决一下文章开头的问题：构建任务中 html 解析错误，出现了一个 Unhandled Rejection，所以我们可以添加一个 `unhandledRejection` 事件监听，直接退出：
```
process.on('unhandledRejection', error => {
  console.error('unhandledRejection', error);
  process.exit(1) // To exit with a 'failure' code
});
```

## 参考链接
[Unhandled Promise Rejections in Node.js](http://thecodebarbarian.com/unhandled-promise-rejections-in-node.js.html)
