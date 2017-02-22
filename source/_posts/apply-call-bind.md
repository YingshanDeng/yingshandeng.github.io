title: JavaScript 中的 apply、call、bind
date: 2017-02-19 16:45:07
tags: [apply, call, bind]
categories: JS
---


JavaScript 中的 apply、call、bind 都可以指定函数执行上下文，本文将简要理解和分析其中的异同。
<!-- more -->

## `call` 和 `apply`
### 语法
```
fun.call(thisArg[, arg1[, arg2[, ...]]])
fun.apply(thisArg[, argsArray])
```
`call` 和 `apply` 方法类似，第一个参数 `thisArg` 指定函数运行时的 `this`，唯一的区别，就是call()方法接受的是若干个参数的列表，而apply()方法接受的是一个包含多个参数的数组。

从语法中可以知道，这两个方法的作用就是调用函数并且指定上下文的 `this`
```
function test(arg1, arg2) {
  console.log(this.x, '--', arg1, '--', arg2);
}

var obj = {
  x: 1
}
test(2, 3) // undefined -- 2 -- 3
test.apply(obj, [2, 3]) // 1 -- 2 -- 3
test.call(obj, 2, 3) // 1 -- 2 -- 3
```

### 使用 `call` 和 `apply` 的例子

### 使用 `call` 方法调用父构造函数
在一个子构造函数中，你可以通过调用父构造函数的 `call` 方法来实现继承，可以参考这篇文章中的例子: [JS继承与原型链](http://objcer.com/2017/02/17/JS-Inheritance-Prototype/)

### `apply` 允许你在某些本来需要写成遍历数组变量的任务中使用内建的函数
例如查找数组中的最大、最小值
```
var numbers = [5, 6, 2, 3, 7];

var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);

// Object Spread Operator
var max = Math.max(...numbers);
var min = Math.min(...numbers);
```
注意：这里并不需要指定 this，所以第一个参数为 null 或者 undefined；其实这里用 `apply` 只是为了方便传参而已，而使用 ES6 的 Object Spread Operator 也可以达到类似的效果

> 但是当心：如果用上面的方式调用 apply, 你很可能会遇到方法参数个数越界的问题. 当你对一个方法传入非常多的参数 (比如超过1W多个参数) 时, 就非常有可能会导致越界问题, 这个临界值是根据不同的 JavaScript 引擎而定的。如果你的参数数组可能非常大, 那么推荐使用下面这种策略来处理: 将参数数组切块后循环传入目标方法:
```
function minOfArray(arr) {
  var min = Infinity;
  var QUANTUM = 32768;

  for (var i = 0, len = arr.length; i < len; i += QUANTUM) {
    var submin = Math.min.apply(null, arr.slice(i, Math.min(i + QUANTUM, len)));
    min = Math.min(submin, min);
  }

  return min;
}

var min = minOfArray([5, 6, 2, 3, 7]);
```

### 在"monkey-patching"中使用 `apply`
> 动态修改已有的方法，称之为猴子补丁

```
var originalfoo = someobject.foo;
someobject.foo = function() {
  //在调用函数前干些什么
  console.log(arguments);
  //像正常调用这个函数一样来进行调用：
  originalfoo.apply(this,arguments);
  //在这里做一些调用之后的事情。
}
```

## `bind`
`bind` 函数同样也是修改函数的 `this` 指向，然后返回一个新的函数，称之为绑定函数。
```
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```
`bind` 的第一个参数将作为它运行时的 this, 之后的一序列参数将会在传递的实参前传入作为它的参数。
```
var obj = {
  x: 12,
  method: function(value1, value2) {
    console.log(this.x, '---', value1, '---', value2)
  }
}

var k = {
  x: 13
}

var kmethod = obj.method.bind(k, 'value1');
kmethod('value2');  // log: 13 "---" "value1" "---" "value2"
```

### 使用 `bind` 快捷调用
1、用 `Array.prototype.slice` 来将一个类似于数组的对象（array-like object）转换成一个真正的数组
```
function test() {
  var slice = Array.prototype.slice;
  var arr = slice.apply(arguments);
  console.log(arr);
}

test(1, 2, 3)
```
用 bind() 可以使这个过程变得简单。在下面这段代码里面，slice 是 Function.prototype 的 call() 方法的绑定函数，并且将 Array.prototype 的 slice() 方法作为 this 的值。这意味着我们压根儿用不着上面那个 apply() 调用了。
```
// same as "slice" in the previous example
var unboundSlice = Array.prototype.slice;
var slice = Function.prototype.call.bind(unboundSlice);

slice(arguments);
```
2、获取数组中的最大、最小值
```
var MAX = Function.prototype.apply.bind(Math.max, null);
MAX([1, 2, 3]);

// 等价于
Math.max.apply(null, [1, 2, 3])
```

### 其他
`call` 和 `apply` 函数都是立即执行的，但是 `bind` 并不是立即执行的，而是返回一个 wrapped apply function。
`bind` 函数内部实现：
```
function bind(func, context) {
    return function() {
        return func.apply(context, arguments);
    };
}
```
看到一个有意思的问题：如果一个函数多次绑定，那么实际上哪一次绑定会生效呢？例如：
```
var obj = {
  x: 0,
  log: function() {
    console.log(this.x)
  }
}

var obj1 = {
  x: 1
}
var obj2 = {
  x: 2
}
var obj3 = {
  x: 3
}

var _log = obj.log.bind(obj1).bind(obj2).bind(obj3);
_log(); // 输出：1
```
显然是第一次绑定生效，其后的绑定都是无效的。原因是 `bind` 函数返回一个闭包，多次绑定就形成了嵌套，显然嵌套在最里面的生效了，也就是第一次绑定。

## 参考链接

[Function.prototype.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
[Function.prototype.apply()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
[Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#Example:_Creating_shortcuts)