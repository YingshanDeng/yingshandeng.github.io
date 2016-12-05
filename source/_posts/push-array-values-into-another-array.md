title: Push array values into another array
date: 2016-12-05 17:51:31
tags: Array
categories: JS
---

> 问题描述：将一个数组的所有元素 push 到另一个数组中
```
var a = [1, 2, 3],
    b = [4, 5, 6];
==>
a = [1, 2, 3, 4, 5, 6]
或者得到一个新的数组
c = [1, 2, 3, 4, 5, 6]
```
<!-- more -->

## 方式一：使用 ES6 的 `spread operator` 特性

```
var a = [1, 2, 3],
	b = [4, 5, 6];
a.push(...b);
console.log(a) // a = [1, 2, 3, 4, 5, 6]
```
上面这段代码经过 babel 编译后，变成：
```
var a = [1, 2, 3],
    b = [4, 5, 6];
a.push.apply(a, b);
```

所以，如果不想用 `spread operator` ES6 这个新的特性的话，也可以如下：
```
var a = [1, 2, 3],
    b = [4, 5, 6];

a.push.apply(a, b);
// 或者
Array.prototype.push.apply(a, b);
```

## 方式二：使用 `concat` 方法
> The concat() method is used to merge two or more arrays. This method does not change the existing arrays, but instead returns a new array.(合并两个或者数组，返回一个新的结果数组)
**Syntax**
var new_array = old_array.concat(value1[, value2[, ...[, valueN]]])
**Parameters**
valueN: Arrays and/or values to concatenate into a new array.

```
var a = [1, 2, 3],
    b = [4, 5, 6];

var c = a.concat(b);
// 或者
var c = [].concat(a, b);

console.log(c) // c = [1, 2, 3, 4, 5, 6]
```

## 方式三：使用数组构造函数 `Array constructor`

```
var a = [1, 2, 3],
    b = [4, 5, 6];
var c = [
        ...a,
        ...b
    ];
//或者
var c = new Array(...a, ...b);
```
上面这段代码经过 babel 编译后，变成：

```
var a = [1, 2, 3],
    b = [4, 5, 6];
var c = [].concat(a, b);

```

