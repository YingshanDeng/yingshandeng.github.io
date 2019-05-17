title: 关于 ES6 Promise
date: 2017-05-07 18:03:15
tags: Promise
categories: JS
---

![](http://cdn.objcer.com/do-you-promise.png)

<!-- more -->

## 串行 Promise
Promise 有一个 `Promise.all` 方法，接收一个 Promise 对象数组作参数，**并行**执行其中的 Promise 对象，当这个数组中的所有 Promise 对象都 `resolve` 或者某一个 Promise 对象 `reject` 时，调用 `then` 方法。

但是，如果需要进行在 A 处理完成之后，再开始 B 的处理，对于这种串行顺序处理，Promise 并没有提供支持。那我们只能自己解决。

### 重复使用多个 `then` 实现
```
function getPromise(name) {
    return new Promise(function(resolve, reject) {
        setTimeout(() => {
            console.log('promise: ', name);
            resolve();
        }, 2000);
    })
}

getPromise('one').then(getPromise.bind(undefined, 'two'));
```
注意：通过 `new` 实例化一个 Promise 的时候，promise 已经开始执行，所以我们都是通过使用一个函数，返回一个 Promise 对象

通过使用 `then` 将两个 Promise 串联起来，使其顺序执行，但是随着串行处理的 Promise 增多，需要不断增加 `then` 方法的调用，在实际使用过程中并不方便。我们可以通过 `for` 循环解决这个问题。

### 通过 `for` 循环实现
```
var ps = [
    getPromise.bind(undefined, 'one'),
    getPromise.bind(undefined, 'two')
]
var promise = Promise.resolve();
ps.forEach((p) => {
    promise = promise.then(p);
});
```
注意：数组中的元素都是返回 Promise 对象的函数

这种处理方式需要一个中间变量 `promise`，通过不断覆盖 `promise` 变量的值，达到 `promise1.then(promise2)` 的累积效果。但是由于多使用了一个中间变量，代码显得不够简洁。

### 通过 `Array.prototype.reduce` 实现
```
var ps = [
    getPromise.bind(undefined, 'one'),
    getPromise.bind(undefined, 'two')
]
ps.reduce((promise, p) => {
    return promise.then(p);
}, Promise.resolve())
```
这种实现串行 Promise 就显得很简洁了吧，在实际开发应用中大多也用此方式 👊
> 关于 reduce 用法，参考 [实现 map, reduce, filter, find 方法](http://objcer.com/2017/03/11/implement-the-array-map-reduce-filter-find-method/#more)

### 定义串行 Promise 函数
在应用开发过程中，许多需要 Promise 串行执行，例如多个网络请求串行时，我们就可以通过如下方式进行，该方法接收一个接收一个函数数组，数组中的函数元素都返回一个 Promise 对象，并将每一个 Promise 对象 `resolve` 的执行结果存储在一个数组中，等所有的 Promise 对象 `resolve` 之后，将结果数组返回。
```
static sequencePromise(tasks) {
    function _recordResults(resultArray, result) {
        resultArray.push(result);
        return resultArray;
    }
    let pushResult = _recordResults.bind(null, []);
    return tasks.reduce((promise, task) => {
        return promise.then(task).then(pushResult);
    }, Promise.resolve())
}
```

## 并行 Promise

### 关于 `Promise.all` 和 `Promise.race`
Promise 的这两个静态方法，都接收一个 iterable 可迭代对象（例如：Promise Array）作为参数，**并行**执行其中的 Promise 实例，结果返回一个 Promise；二者的区别是：
- `Promise.all`: 可迭代对象中的所有 Promise 都 resolve 之后 resolve；或者在任一 Promise 被 reject 后 reject
- `Promise.race`: 可迭代对象中的第一个 Promise 被 resolve 或者 reject 后，进行 resolve 或者 reject

看如下的例子：
有如下三个 Promise 实例
```
var p1 = new Promise((resolve, reject) => {
  setTimeout(() => resolve('p1'), 2000)
});
var p2 = new Promise((resolve, reject) => {
  setTimeout(() => reject('p2'), 1000)
});
var p3 = new Promise((resolve, reject) => {
  setTimeout(() => resolve('p3'), 500)
});
```

① `Promise.all`：所有 Promise 都 resolve 之后 resolve
```
Promise.all([p1, p3]).then((value) => {
    console.log('--resolve--: ', value)
}).catch((err) => {
    console.log('--reject--: ', err)
})
// 输出：--resolve--:  ["p1", "p3"]
```
② `Promise.all`：任一 Promise 被 reject 后 reject
```
Promise.all([p1, p2, p3]).then((value) => {
    console.log('--resolve--: ', value)
}).catch((err) => {
    console.log('--reject--: ', err)
})
// 输出：--reject--:  p2
```
① `Promise.race`：第一个 Promise 被 resolve 或者 reject 后，进行 resolve 或者 reject
```
Promise.race([p1, p2, p3]).then((value) => {
    console.log('--resolve--: ', value)
}).catch((err) => {
    console.log('--reject--: ', err)
})
// 输出：输出：--reject--:  p3
```

### 并行 Promise
对于：*并行执行 Promises 等待所有的 Promise 都完成执行（不管是 resolve 还是 reject）才结束。*这种情形使用 `Promise.all` 并不能满足，因为 `Promise.all` 在任一 Promise 被 reject 后 reject，这种情形需要对 Promise 进行一些处理后，再使用 `Promise.all`
> 参考：[Wait until all ES6 promises complete, even rejected promises](https://stackoverflow.com/questions/31424561/wait-until-all-es6-promises-complete-even-rejected-promises?answertab=votes#tab-top)

```
function parallelPromise(tasks) {
    function reflect(promise) {
         return promise.then(function(v){ return {v:v, status: "resolved" }},
                            function(e){ return {e:e, status: "rejected" }});
    }
    return Promise.all(tasks.map(reflect));
}

var p1 = new Promise((resolve, reject) => {
  setTimeout(() => resolve('p1'), 2000)
});
var p2 = new Promise((resolve, reject) => {
  setTimeout(() => reject('p2'), 1000)
});
var p3 = new Promise((resolve, reject) => {
  setTimeout(() => resolve('p3'), 500)
});

parallelPromise([p1, p2, p3]).then((results) => {
    var success = results.filter(x => x.status === "resolved");
    var fail = results.filter(x => x.status === "rejected");
    console.log('--success--: ', JSON.stringify(success));
    console.log('--fail--: ', JSON.stringify(fail));
})
```
执行结果：
```
--success--:  [{"v":"p1","status":"resolved"},{"v":"p3","status":"resolved"}]
--fail--:  [{"e":"p2","status":"rejected"}]
```

## 可取消的 Promise
Promise 并没有提供 `cancel` 的接口，但是我们可以对其进行封装实现 Promise 可取消(cancelable).
> 参考自：[isMounted is an Antipattern](https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html)

```
const makeCancelable = (promise) => {
    let hasCanceled_ = false;

    const wrappedPromise = new Promise((resolve, reject) => {
        promise.then((val) =>
            hasCanceled_ ? reject({isCanceled: true}) : resolve(val)
        );
        promise.catch((error) =>
              hasCanceled_ ? reject({isCanceled: true}) : reject(error)
        );
    });

    return {
        promise: wrappedPromise,
        cancel() {
          hasCanceled_ = true;
        },
    };
};
```
测试例子🌰：
```
var cancelablePromise = makeCancelable(new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('ok');
    }, 3000)
}));

cancelablePromise
    .promise
    .then((value) => {
        console.log('---resolve---', value);
    })
    .catch((reason) => {
        console.log('---cancel---', reason.isCanceled);
    });

// cancelablePromise.cancel()
```

