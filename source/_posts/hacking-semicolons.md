title: JavaScript 中何时加分号❓
date: 2017-10-13 10:06:47
tags:
categories: JS
---

在阅读 medium 上的文章 [JavaScript: async/await with forEach()](https://codeburst.io/javascript-async-await-with-foreach-b6ba62bbf404) 时，作者用了如下一个例子来阐述：async/await 和 forEach 之间的问题。但是仔细琢磨这段代码，发现不对劲啊 😤 明显和我之前的认知不同啊。本文将分析一下其中的蹊跷。

> 关于 async/await 和 forEach，我在 [当 async/await 遇上 forEach](https://objcer.com/2017/10/12/async-await-with-forEach/) 文中进行了详细分析，读者若对此不熟悉，请先前往了解 👉

```
const waitFor = (ms) => new Promise(r => setTimeout(r, ms))
[1, 2, 3].forEach(async (num) => {
  await waitFor(50)
  console.log(num)
})
console.log('Done')
```

<!-- more -->

## 问题分析
当 async/await 遇上 forEach，无非就是将回调的异步函数，全部并行执行了，而非我们期望的串行执行。那么对于这段代码，正常的输出结果是：输出：`Done`，间隔 50 毫秒，输出：1，2，3；但是实际却只输出了 `Done`，所以我们可以断定这段代码有问题。

### 考虑箭头函数的问题
一开始我把目光放到箭头函数上，箭头函数 `waitFor` 返回一个 Promise 对象， 把箭头函数返回改造成如下，发现，执行结果正确了
```
var waitFor = (ms) => {return new Promise(r => setTimeout(r, ms))}
[1, 2, 3].forEach(async (num) => {
  await waitFor(50)
  console.log(num)
})
console.log('Done')
```

然后，在 [MDN - Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) 发现：
```
// Parenthesize the body of function to return an object literal expression:
params => ({foo: bar})
```

指箭头函数返回对象字面量时，需要用括号包起来。对于如下函数：
```
var square = function(n) {
    return {
        square: n * n
    }
}
```

可以改写成箭头函数如下：
```
var square = n => { square: n * n }
```

此时调用 `square` 函数，会发现，无论传入什么参数都没有任何输出，为啥呢？

JavaScript 在解析 `square` 函数时，遇到 `{ square: n * n }`，并不是认为其是一个对象字面量，而把它解析成 `Labelled Block Statement`，label 为 `square`，statement 为 `n * n`；由于没有返回值，执行 `square` 函数会一直返回 undefined，如果返回 `n * n` 那么代码也可正常执行
```
var square = n => { square: return n * n }
```

显然这样的代码太奇怪了，最简单的方式是用括号将对象字面量包起来，这样，JavaScript 在解析的时候遇到括号，就不会将 `{ square: n * n }` 认为是 `Labelled Block Statement` 了，而是对象字面量，如下：
```
var square = n => ({ square: n * n })
```

但是，但是，我们分析的这段代码中 `waitFor` 箭头函数返回的是 Promise 对象，并非对象字面量，所以并不是箭头函数的锅啊。

### 分号
我们看到如下代码：
```
var a = {'b': [1, 2, 3]}
['b'].forEach(v => {console.log(v)})  // 输出： 1, 2, 3

var a = {'b': 1}
['b'].forEach(v => {console.log(v)}) // 出错：Uncaught TypeError: {(intermediate value)}.b.forEach is not a function
```

显然 JavaScript 在代码解析的时候，第一行和第二行没有分开啊，我们可以加上分号，这样代码就正常执行了。
我们分析的这段代码，在 `waitFor` 箭头函数后加上分号，代码也就正确执行了
```
const waitFor = (ms) => new Promise(r => setTimeout(r, ms));
[1, 2, 3].forEach(async (num) => {
  await waitFor(50)
  console.log(num)
})
console.log('Done')
```

这是为啥呢？似乎在我们的印象中，JavaScript 是可以忽略分号的，莫非有特殊情况。在知乎问题： [JavaScript 语句后应该加分号么？](https://www.zhihu.com/question/20298345/answer/49551142) 中，我们发现：
> 👉 **真正会导致上下行解析出问题的 token 有 5 个：括号，方括号，正则开头的斜杠，加号，减号。**我还从没见过实际代码中用正则、加号、减号作为行首的情况，所以总结下来就是一句话：**一行开头是括号或者方括号的时候加上分号就可以了，其他时候全部不需要。**

也即是在 `+ - [ ( /` 这五个字符开头时，需要在上一行或者当前行首加上分号，避免 JavaScript 上下行解析出错。

## 问题总结
medium 的作者不小心踩了一个坑，少写了一个分号，导致 JavaScript 上下文解析出错，写了一个不恰当的例子，通过一步步分析，还是找到了问题的本质，也算是涨姿势了😋

## 参考链接
[Returning Object Literals from Arrow Functions in JavaScript](https://blog.mariusschulz.com/2015/06/09/returning-object-literals-from-arrow-functions-in-javascript)
