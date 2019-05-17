title: 当 async/await 遇上 forEach
date: 2017-10-12 16:28:38
tags: [async, await]
categories: JS
---


在文章 [ES7 中的 async await](https://objcer.com/2017/10/11/Async-Await/) 中介绍了 async/await。本文将分析介绍当 async/await 遇上 `forEach` 出现的一些问题和解决方案。

![](http://cdn.objcer.com/1-pWGJIKats-zuumA3RQNEWQ.jpeg)

<!-- more -->

## 问题描述
```
var getNumbers = () => {
  return Promise.resolve([1, 2, 3])
}
var multi = num => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (num) {
        resolve(num * num)
      } else {
        reject(new Error('num not specified'))
      }
    }, 1000)
  })
}

async function test () {
  var nums = await getNumbers()
  nums.forEach(async x => {
    var res = await multi(x)
    console.log(res)
  })
}

test()
```

在这个例子中，通过 `forEach` 遍历的将每一个数字都执行 `multi` 操作。代码执行的结果是：1 秒后，一次性输出1，4，9。这个结果和我们的预期有些区别，我们是希望每间隔 1 秒，然后依次输出 1，4，9；所以当前代码应该是并行执行了，而我们期望的应该是串行执行。

## 问题分析

### JavaScript 中的循环数组遍历
在 JavaScript 中提供了如下四种循环遍历数组元素的方式：
- `for`
  这是循环遍历数组元素最简单的方式
  ```
  for(i = 0; i < arr.length; i++) {
    console.log(arr[i]);
  }
  ```
- `for-in`
  `for-in` 语句以任意顺序遍历一个对象的可枚举属性，对于数组即是数组下标，对于对象即是对象的 key 值。注意 `for-in` 遍历返回的对象属性都是字符串类型，即使是数组下标，也是字符串 "0", "1", "2" 等等。*[不推荐使用 `for-in` 语句]*
  ```
  for (var index in myArray) {
    console.log(myArray[index]);
  }
  ```

- `forEach`
  `forEach` 方法用于调用数组的每个元素，并将元素传递给回调函数；注意在回调函数中无法使用 `break` 跳出当前循环，也无法使用 `return` 返回值
  ```
  myArray.forEach(function (value) {
    console.log(value);
  });
  ```

- `for-of`
  `for-of` 语句为各种 collection 集合对象专门定制的，遍历集合对象的属性值，注意和 `for-in` 的区别
  ```
  for (var value of myArray) {
    console.log(value);
  }
  ```

### 分析问题
在本例中 `forEach` 的回调函数是一个异步函数，异步函数中包含一个 `await` 等待 Promise 返回结果，我们期望数组元素串行执行这个异步操作，但是实际却是并行执行了。

`forEach` 的 polyfill 参考：[MDN-Array.prototype.forEach()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)，简单点理解：

```
Array.prototype.forEach = function (callback) {
  // this represents our array
  for (let index = 0; index < this.length; index++) {
    // We call the callback for each entry
    callback(this[index], index, this)
  }
}
```

相当于 `for` 循环执行了这个异步函数，所以是并行执行，导致了一次性全部输出结果：1，4，9
```
async function test () {
  var nums = await getNumbers()
//   nums.forEach(async x => {
//     var res = await multi(x)
//     console.log(res)
//   })

  for(let index = 0; index < nums.length; index++) {
    (async x => {
      var res = await multi(x)
      console.log(res)
    })(nums[index])
  }
}
```

## 解决问题

### 方式一
我们可以改造一下 `forEach`，确保每一个异步的回调执行完成后，才执行下一个
```
async function asyncForEach(array, callback) {
  for (let index = 0; index < array.length; index++) {
    await callback(array[index], index, array)
  }
}

async function test () {
  var nums = await getNumbers()
  asyncForEach(nums, async x => {
    var res = await multi(x)
    console.log(res)
  })
}
```

### 方式二
使用 `for-of` 替代 `for-each`。

`for-of` 可以遍历各种集合对象的属性值，要求被遍历的对象需要实现迭代器 (iterator) 方法，例如 `myObject[Symbol.iterator]()` 用于告知 JS 引擎如何遍历该对象。一个拥有 `[Symbol.iterator]()` 方法的对象被认为是可遍历的。

```
var zeroesForeverIterator = {
  [Symbol.iterator]: function () {
    return this;
  },
  next: function () {
    return {done: false, value: 0};
  }
};
```

如上就是一个最简单的迭代器对象；`for-of` 遍历对象时，先调用遍历对象的迭代器方法 `[Symbol.iterator]()`，该方法返回一个迭代器对象(迭代器对象中包含一个 `next` 方法)；然后调用该迭代器对象上的 `next` 方法。

每次调用 `next` 方法都返回一个对象，其中 `done` 和 `value` 属性用来表示遍历是否结束和当前遍历的属性值，当 `done` 的值为 `true` 时，遍历就停止了。

```
for (VAR of ITERABLE) {
  STATEMENTS
}
```
等价于：
```
var $iterator = ITERABLE[Symbol.iterator]();
var $result = $iterator.next();
while (!$result.done) {
  VAR = $result.value;
  STATEMENTS
  $result = $iterator.next();
}
```

由此可以知道 `for-of` 和 `forEach` 遍历元素时处理的方式是不同的。使用 `for-of` 替代 `for-each` 后代码为：
```
async function test () {
  var nums = await getNumbers()
  for(let x of nums) {
    var res = await multi(x)
    console.log(res)
  }
}
```

## 参考链接
[Understand async/await better](https://codeburst.io/understand-async-await-better-7a03aeba60fe)
[ES6 In Depth: Iterators and the for-of loop](https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/)