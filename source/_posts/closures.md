title: 闭包（closures）
date: 2017-03-19 21:58:11
tags: 闭包
categories: JS
---


JS 中闭包是一种特殊的对象。 **它由两部分构成：函数，以及创建该函数的环境。环境由闭包创建时在作用域中的任何局部变量组成。**简单的理解：在一个函数中创建了另一个函数，也就创建了一个闭包。


<!-- more -->

👇下面将通过一些闭包使用例子来理解闭包这个概念。

## 闭包概念
```
function init() {
    var name = 'Cony';
    return function() {
        console.log(name)
    }
}

var displayName = init();
displayName();
```
在这个例子中，`init` 函数执行后返回的匿名函数 `displayName` 就是一个闭包，由 `displayName` 函数和闭包创建时存在的 `Cony` 字符串形成。

其中有两点需要注意：
- 闭包可以获取引用到创建时所在环境的局部变量
- `init` 函数执行后，其中的局部变量并不会马上被释放，所以在稍后闭包函数执行过程中可以被引用到（这一点可能和其他编程语言有点区别，例如C语言中，函数执行完毕，其中的局部变量就被释放了）

## 闭包引用创建环境中的局部变量
```
function init() {
    var num = 12;
    var log = function() {
        console.log(num)
    }
    num ++;
    return log;
}

init()(); // 13
```
这个例子说明闭包函数在使用局部变量时是通过引用传递，而非值传递。

```
function init() {
    var log = function() {
        console.log(num)
    }
    var num = 12;
    return log;
}

init()(); // 12
```
这个例子说明，闭包函数可以使用创建时所在环境的所有局部变量，即便是在其后声明定义的。
> 这个例子中涉及 variable hoisting（变量提升），可以参考 [Variable and Function Hoisting in ES2015](https://bitsofco.de/variable-and-function-hoisting-in-es2015/)

## 在循环中创建闭包
```
function buildList(list) {
    var result = [];
    for (var i = 0; i < list.length; i++) {
        var item = 'item' + i;
        result.push( function() { console.log(item + ' ' + list[i])} );
    }
    return result;
}

function testList() {
    var fnlist = buildList([1,2,3]);
    // Using j only to help prevent confusion -- could use i.
    for (var j = 0; j < fnlist.length; j++) {
        fnlist[j]();
    }
}
testList()

```
这个例子中， `buildList` 函数接收一个数组，然后为每一个数组元素创建一个闭包函数，返回闭包函数数组。期望中，`testList` 函数执行后，输出：
```
item0 1
item1 2
item2 3
```
但是上面这段代码却重复输出3次：`item2 undefined`，这是为什么呢？🤔
*分析一下：*
在 `buildList` 函数中的 `for` 循环为 `result` 数组添加三个匿名函数，也就是闭包函数，这三个闭包函数创建时所在的环境相同，共享其中的局部变量（`item`，`i` 和 `list`）。值得注意的是，在函数 `buildList` 执行结束后， 局部变量 `item` 的值变成了 `item2`，`i` 的值已经变成 3，所以 `list[i]` 的值就是 `undefined`。所以在 `testList` 函数执行闭包函数 `fnlist[j]()` 时，每一次的输出结果都是相同的 `item2 undefined`。

那么要如何修改，才能得到我们期望中的结果呢？方式有好多种，简单罗列两种。
1、使用 ES6 `let` 代替 `var`
```
// 在 buildList 函数中，将其中的两处 var 替换成 let
for (var i = 0; i < list.length; i++) {
    var item = 'item' + i;
    result.push( function() { console.log(item + ' ' + list[i])} );
}
-->
for (let i = 0; i < list.length; i++) {
    let item = 'item' + i;
    result.push( function() { console.log(item + ' ' + list[i])} );
}
```
2、通过闭包解决
```
function buildList(list) {
    var result = [];
    for (var i = 0; i < list.length; i++) {
        (function() {
            var index = i;
            var item = 'item' + index;
            result.push( function() {console.log(item + ' ' + list[index])} );
        })();
    }
    return result;
}
```
我们知道，闭包函数可以引用创建时所在环境的局部变量，这里通过闭包，`for` 循环中每一个闭包引用的 `index` 都不同，都是单独的，这样，代码就如我们预期的结果运行。

## 用闭包模拟私有属性和私有方法
在 JS 中并不支持私有属性和方法，但是通过闭包可以模拟实现。通过闭包来定义公共接口，并隐藏私有方法和变量，这也称之为模块模式（module pattern）
```
var Counter = (function() {
    var privateCounter = 0;
    function _changeBy(val) {
        privateCounter += val;
    }
    return {
        increment: function() {
            _changeBy(1);
        },
        decrement: function() {
            _changeBy(-1);
        },
        value: function() {
            return privateCounter;
        }
    }
})();
```
其中 `privateCounter` 和 `_changeBy` 分别为私有变量和私有方法，无法通过 `Counter` 直接访问；而 `increment` `decrement` `value` 均为公共接口。

## 思考题
在 [学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html) 这篇文章最后提到两个例子，引用到此处，以下两段代码分别输出什么，其中也涉及到 `this` 关键字的问题。
```
var name = "The Window";
var object = {
    name: "My Object",
    getNameFunc: function() {
        return function() {
            return this.name;
        }
    }
};
console.log(object.getNameFunc()());
```

```
var name = "The Window";
var object = {
    name: "My Object",
    getNameFunc: function() {
        var that = this;
        return function() {
            return that.name;
        }
    }
};
console.log(object.getNameFunc()());
```

## 参考链接
[JavaScript Closures for Beginners](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work?answertab=votes#tab-top)