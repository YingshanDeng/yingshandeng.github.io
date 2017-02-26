title: 探究 JS 中的浅拷贝和深拷贝
tags:
---

<!-- more -->
JS 中数据类型可划分为
- Primitive Data Types(原始类型): **String, Number, Boolean, Null, Undefined**
- Composite(reference) Data Types(引用类型): **Object, Array**

ECMAScript 有 5 种原始类型（primitive type），即 Undefined、Null、Boolean、Number 和 String。其中 Undefined 类型和 Null 类型都只有一个值，分别是 undefined 和 null。
> 通过 `typeof` 运算符可以检查一个变量或者值的类型，例如变量是 String 类型，那么 `typeof` 得到的结果就是 `string`。但是有一个例外是 `typeof null` 的结果是 `object`。那为什么 `typeof` 运算符对于 null 值会返回 "Object"。这实际上是 JavaScript 最初实现中的一个错误，然后被 ECMAScript 沿用了。现在，null 被认为是对象的占位符，从而解释了这一矛盾，但从技术上来说，它仍然是原始值。

对于原始类型和引用类型的区别是传值方式，前者是传 value，或者是传 reference。

## 浅拷贝 shallow clone
```
function shallowClone(obj) {
    if(!obj) return obj; // null or undefined
    var clone = {}
    for(var prop in obj) {
        if(obj.hasOwnProperty(prop)) {
            clone[prop] = obj[prop];
        }
    }
    return clone;
}

var obj = {a: 1, b: 2};
var _clone = shallowClone(obj)
obj === _clone // false
```
在 ES6 中提供了其他可以浅拷贝的方式：
1、`Object.assign`
```
const existing = { a: 1, b: 2, c: 3 };
const clone = Object.assign({}, existing);
```
2、Object rest/spread destructuring
```
const existing = { a: 1, b: 2, c: 3 };

const { ...clone } = existing;
```
> 这种方式可能浏览器支持并不是很好，在 babel 编译后的代码中我们注意到：
```
function _objectWithoutProperties(obj, keys) {
    var target = {};
    for (var i in obj) {
        if (keys.indexOf(i) >= 0) continue;
        if (!Object.prototype.hasOwnProperty.call(obj, i)) continue;
        target[i] = obj[i];
    }
    return target;
}
// 示例一:
// 编译前
const existing = { a: 1, b: 2, c: 3 };
const { ...clone } = existing;
// 编译后
var existing = { a: 1, b: 2, c: 3 };
var clone = _objectWithoutProperties(existing, []);

// 示例二:
// 编译前
const existing = { a: 1, b: 2, c: 3 };
const { a, ...clone } = existing;
// 编译后
var existing = { a: 1, b: 2, c: 3 };
var a = existing.a;
var clone = _objectWithoutProperties(existing, ['a']);
```

如果拷贝对象中的值都是原始类型，那么使用浅拷贝进行对象拷贝没什么问题，但是如果对象中包含了引用类型，进行浅拷贝，就可能有潜在的问题。

## 深拷贝 deep clone
