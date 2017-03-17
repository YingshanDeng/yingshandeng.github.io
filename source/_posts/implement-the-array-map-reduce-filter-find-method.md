title: '实现 map, reduce, filter, find 方法'
date: 2017-03-11 16:22:26
tags: [map, reduce, filter, find]
categories: JS
---


在对数组进行操作中，最常遇到的操作就是遍历数组，进行相关的数据处理，最简单的做法是使用循环，`for` 或者 `while`，然而这样的写法并不算是最佳的（[JAVASCRIPT WITHOUT LOOPS](http://jrsinclair.com/articles/2017/javascript-without-loops/)）；在 JavaScript 中，提供了 `map`，`reduce`，`reduceRight`，`filter`，`find` 等方法更加灵活的实现对数组数据的操作。

本文将对 `map`，`reduce`，`reduceRight`，`filter`，`find` 方法的用法进行简单介绍，并各自手动对其实现。其中大部分内容都是阅读参考 MDN 文档而来。

<!-- more -->

## `map`
`map` 方法创建一个新数组，其结果是该数组中的每个元素调用一个提供的回调函数。
```
var new_array = arr.map(callback[, thisArg])
```
- `callback`: 使用三个参数 `currentValue`, `index`, `array`
- `thisArg`: 可选参数，指定执行 `callback` 时的 `this` 值

手动实现该方法：
```
Array.prototype._map = function(callback /*, thisArg*/) {
    if (this == null) {
        // undefined == null
        throw new TypeError('null or undefined');
    }
    if (typeof callback !== 'function') {
        throw new TypeError('callback is not a function');
    }
    var oldArr = Object(this);
    var len = oldArr.length >>> 0;
    // or: var len = +oldArr.length || 0;
    var thisArg;
    if (arguments.length > 1) {
        thisArg = arguments[1];
    }
    var newArr = new Array(len);
    var k = 0;
    while (k < len) {
        if (k in oldArr) {
            newArr[k] = callback.call(thisArg, oldArr[k], k, oldArr);
        }
        k ++;
    }
    return newArr;
}

var array = [1, 2, 3, 4, 5];
array._map(function(value, index, arr) {
    console.log('-', value, '-', index, '-', arr);
    return value * 2;
})
```

**📌注意**
1、在上述的代码中我们注意到如下的代码：
```
var t = Object(this);
var len = t.length >>> 0;
```
其中 `t.length >>> 0` 是什么用法呢？

`>>>`: 无符号右移位赋值(Unsigned right shift assignment)
`t.length >>> 0` 此处的作用是，可以将一个 non-Number 转换成 non-negative Number.

对于 `var t = Object(this);` 如果 `this` 为 `null` 或者 `undefined`，那么返回的就是一个空的对象，那么这时 `t.length` 获取对象的 `length` 属性就会返回 `undefined`；而 `t.length >>> 0` 就是为了纠正这个潜在的问题。

其实了解了为啥需要 `t.length >>> 0` 的原因后，我们其他可以换一种更浅显易懂的解决方式：
```
// Cast this.length to a number, or use 0 if this.length is
// NaN/undefined (evaluates to false)
var len = +this.length || 0;
```
*参考链接：*
[why-are-the-mdc-prototype-functions-written-this-way](http://stackoverflow.com/questions/5969124/why-are-the-mdc-prototype-functions-written-this-way)
[what-is-the-javascript-operator-and-how-do-you-use-it](http://stackoverflow.com/questions/1822350/what-is-the-javascript-operator-and-how-do-you-use-it)

2、为对象添加方法有两种方式，此处是直接在数组的原型(`Array.prototype`)中添加方法；还有另一种方式：通过 `Object.defineProperty` 为对象定义一个方法。建议使用后者，因为直接在原型中添加方法，会有一个缺陷就是，添加的方法是可枚举的，而通过 `Object.defineProperty` 可以配置方法的可枚举属性。


## `reduce` 和 `reduceRight`

1、`reduce` 方法对累加器和数组的每个值 (从左到右)应用一个函数，以将其减少为单个值。
```
arr.reduce(callback[, initialValue])
```
- `callback`: 使用四个参数 `accumulator`, `currentValue`, `currentIndex`, `array`
- `initialValue`: 可选参数

例子：数组求和
```
var sum = [1, 2, 3, 4].reduce((acc, cur) => acc + cur, 0) // 10
```

手动实现该方法：
```
Array.prototype._reduce = function(callback /*, initialValue*/) {
    if (this == null) {
        // undefined == null
        throw new TypeError('null or undefined');
    }
    if (typeof callback !== 'function') {
        throw new TypeError('callback is not a function');
    }

    var oldArr = Object(this);
    var len = oldArr.length >>> 0;
    var initialValue, k=0;

    if(arguments.length >= 2) {
        initialValue = arguments[1];
    } else {
        while(k < len && !(k in oldArr)) k++;
        if(k >= len) {
            throw new TypeError( 'Reduce of empty array ' +
            'with no initial value' );
        }
        initialValue = oldArr[k++];
    }

    while(k < len) {
        if(k in oldArr) {
            initialValue = callback(initialValue, oldArr[k], k, oldArr);
        }
        k++;
    }
    return initialValue;
}

var array = [1, 2, 3, 4, 5];
// var array = []
array._reduce(function(acc, cur, index, arr) {
    console.log('-', acc, '-', cur, '-', index);
    return acc + cur;
}, 100)
```

2、`reduceRight` 方法和 `reduce` 类似，只不过遍历值的顺序和 `reduce` 方法相反，从右向左。

手动实现该方法：
```
Array.prototype._reduceRight = function(callback /*, initialValue*/) {
    if (this == null) {
        // undefined == null
        throw new TypeError('null or undefined');
    }
    if (typeof callback !== 'function') {
        throw new TypeError('callback is not a function');
    }

    var oldArr = Object(this);
    var len = oldArr.length >>> 0;
    var initialValue, k = len - 1;

    if(arguments.length >= 2) {
        initialValue = arguments[1];
    } else {
        while(k >= 0 && !(k in oldArr)) k--;
        if(k < 0) {
            throw new TypeError( 'Reduce of empty array ' +
            'with no initial value' );
        }
        initialValue = oldArr[k--];
    }

    while(k >= 0) {
        if(k in oldArr) {
            initialValue = callback(initialValue, oldArr[k], k, oldArr);
        }
        k--;
    }
    return initialValue;
}

var array = [1, 2, 3, 4, 5];
// var array = []
array._reduceRight(function(acc, cur, index, arr) {
    console.log('-', acc, '-', cur, '-', index);
    return acc + cur;
}, 100)
```

## `filter`
`filter` 方法使用指定的函数测试所有元素，并创建一个包含所有通过测试的元素的新数组。
```
var new_arrary = arr.filter(callback[, thisArg])
```
- `callback`: 使用三个参数 `currentValue`, `index`, `array`，返回 `true` 表示保留该值，`false` 表示排除该值
- `thisArg`: 可选参数，指定执行 `callback` 时的 `this` 值

手动实现该方法：
```
Array.prototype._filter = function(callback /*, thisArg*/) {
    if (this == null) {
        // undefined == null
        throw new TypeError('null or undefined');
    }
    if (typeof callback !== 'function') {
        throw new TypeError('callback is not a function');
    }

    var oldArr = Object(this);
    var len = oldArr.length >>> 0;
    var newArr = [];
    var thisArg = arguments.length >= 2 ? arguments[1] : void 0;
    var k = 0;
    while(k < len) {
        if(k in oldArr) {
            var val = oldArr[k];
            if(callback.call(thisArg, val, k, oldArr)) {
                newArr.push(val);
            }
        }
        k++;
    }
    return newArr;
}
```

## `find`
`find` 方法返回数组中满足提供的测试函数的第一个元素的值。否则返回 undefined。
```
arr.find(callback[, thisArg])
```
- `callback`: 使用三个参数 `currentValue`, `index`, `array`
- `thisArg`: 可选参数，指定执行 `callback` 时的 `this` 值

手动实现该方法：
```
Array.prototype._find = function(callback /*, thisArg*/) {
    if (this == null) {
        // undefined == null
        throw new TypeError('null or undefined');
    }
    if (typeof callback !== 'function') {
        throw new TypeError('callback is not a function');
    }

    var oldArr = Object(this);
    var len = oldArr.length >>> 0;
    var thisArg = arguments.length >= 2 ? arguments[1] : void 0;
    var k = 0;
    while(k < len) {
        if(k in oldArr) {
            var val = oldArr[k];
            if(callback.call(thisArg, val, k, oldArr)) {
                return val;
            }
        }
        k++;
    }
    return undefined;
}
```

> 其中 `void 0` 可参考: [关于 'void 0' 和 'javascript:void(0)'❓](http://objcer.com/2017/03/12/What-does-void-0-mean/)




