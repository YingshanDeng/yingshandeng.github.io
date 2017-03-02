title: "JavaScript typeof vs instanceof\uD83E\uDD14"
date: 2017-02-28 21:23:07
tags: [typeof, instanceof, constructor]
categories: JS
---

*问题引入：*
```
var str = 'foo';
str instanceof String // false
typeof str === 'string' // true
```
那么 JavaScript 中 `instanceof` 和 `typeof` 有啥区别呢？

<!-- more -->

## `typeof`
用法：`typeof operand`；是个一元运算符，就像 `!` 逻辑非运算符，它会返回以字符串表示操作数的类型。
**typeof一般只能返回如下几个结果：number, boolean, string, function（函数）, object（null，数组，对象）, undefined。**
```
// primitive type
typeof 37 === 'number';
typeof "foo" === 'string';
typeof true === 'boolean';
typeof undefined === 'undefined';
// non-primitive type
typeof {a:1} === 'object';
typeof new Date() === 'object';
// 注意
typeof new Boolean(true) === 'object';
typeof new Number(1) ==== 'object';
typeof new String("abc") === 'object';
// 函数
typeof function(){} === 'function';
typeof Math.sin === 'function';
```
**📌注意**
1、关于 `typeof` 有一处比较特殊 `typeof null === 'object'`
2、对于数组 `typeof [1, 2, 4] === 'object';`
所以使用 `typeof` 并不能区分是 `Object` 还是 `Array` 类型，如果要进行区分，可以通过以下几种方法：
- `Array.isArray`
- `Object.prototype.toString.call`
```
var arr = [], obj = {};
Object.prototype.toString.call(arr) === '[object Array]' // true
Object.prototype.toString.call(obj) === '[object Object]' // true
```
- ❗️~~`instanceof`~~
**注意：** 使用 `instanceof` 判断时要特别注意：
```
var arr = [], obj = {};
// 以下两行代码都是返回 true
arr instanceof Array // true
arr instanceof Object // true

obj instanceof Object // true

// 判断是 `Array` 类型
function _isArray(arr) {
    return arr instanceof Array;
}
// 判断是 `Object` 类型
function _isObject(obj) {
    return obj instanceof Object && !(obj instanceof Array);
}
```

3、对正则表达式字面量的类型判断在某些浏览器中不符合标准：
```
typeof /s/ === 'function'; // Chrome 1-12 , 不符合 ECMAScript 5.1
typeof /s/ === 'object'; // Firefox 5+ , 符合 ECMAScript 5.1
```

## `constructor`
`constructor` 是所有对象原型对象中的一个属性，其指向创建该对象实例的构造函数。
1、对于原始类型：
```
var str = '', num = 1;
str.constructor === String // true
num.constructor === Number // true
```
2、对于引用类型：
```
// 注意此处括号不能省略
({}).constructor === Object // true
[].constructor === Array // true
```
3、对于自定义对象：
```
function Person(name) {
  this.name = name;
}

var tom = new Person('Tom');
// tom.__proto__.constructor 等价于 tom.constructor
tom.constructor === Person; //true

// 修改 Person 的原型对象，导致 Person 的原型对象中的 `constructor` 发生变化，变成了 Object
Person.prototype = {}
var jack = new Person('Jack');
jack.constructor === Person // false
jack.constructor === Object
```
**📌注意：**对象的 `constructor` 属性返回的是一个构造函数的引用，但是由于 `constructor` 是原型对象中的一个属性，所以当修改原型对象时，可能就会导致 `constructor` 发生变化。

## `instanceof`
`instanceof` 用法：`object instanceof constructor`；是个二元运算符，用于判断一个**引用类型**对象，**在其原型链中**，是否存在一个原型的构造函数为 `constructor`。
所以对于上一章节中的例子：
```
tom instanceof Person; //true
// 因为 Person.prototype 是 Object，所以在 Object 存在于 tom 的原型链中
tom instanceof Object; //true
```
**📌注意：**对于原始类型，一直返回 `false`。
```
// 此时 str num 都是原始类型
var str = 'foo';
str instanceof String // false
var num = 1;
num instanceof Number // false

// 此时 str num 都是引用类型
var str = new String('foo');
// str instanceof String // true
var num = new Number(1);
num instanceof Number // true
```

## 总结
1、对于文章开头案例，在 JavaScript 中的字符串可能是字符串字面量（primitive type），也可能是字符串对象（non-primitive type），所以在判断一个变量是否是字符串时，可以使用如下方法：
```
function isString(s) {
    return typeof(s) === 'string' || s instanceof String;
}
```
2、何时用 `typeof`，何时用 `instanceof`
- 对于原始类型(primitive type)，使用 `typeof`
- 对于判断一个对象是否是其父类型的实例，使用 `instanceof`
- 对于原始类型，使用 `instanceof` 一直返回 `false`
- `typeof` 一般只能返回如下几个结果：number, boolean, string, function（函数）, object（null，数组，对象）, undefined

## 参考链接
[Understanding typeof, instanceof and constructor in JavaScript](http://skilldrick.co.uk/2011/09/understanding-typeof-instanceof-and-constructor-in-javascript/)
[Why does instanceof return false for some literals?](http://stackoverflow.com/questions/203739/why-does-instanceof-return-false-for-some-literals)

