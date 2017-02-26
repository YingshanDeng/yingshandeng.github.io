title: JS 属性的可枚举性
date: 2017-02-22 17:56:58
tags: [枚举, enumerable, 属性描述符, 实例属性, 原型属性]
categories: JS
---


可枚举属性是指那些内部 “可枚举” 标志（`enumerable`）设置为 `true` 的属性，对于通过直接的赋值和属性初始化的属性，该标识值默认为即为 `true`，对于通过 `Object.defineProperty` ，`Object.create` 等定义的属性，该标识值默认为 `false`。可枚举的属性可以通过 `for...in` 循环进行遍历（除非该属性名是一个 `Symbol`）。

<!-- more -->

## 属性描述符(Properties' descriptor)
一个属性描述符是一个记录，由下面属性当中的某些组成的：
- value // 该属性的值(仅针对数据属性描述符有效)
- writable // 当且仅当属性的值可以被改变时为true。(仅针对数据属性描述有效)
- configurable // 当且仅当指定对象的属性描述可以被改变或者属性可被删除时，为true。
- enumerable // 当且仅当指定对象的属性可以被枚举出时，为 true。
- get // 获取该属性的访问器函数（getter）。如果没有访问器， 该值为undefined。(仅针对包含访问器或设置器的属性描述有效)
- set // 获取该属性的设置器函数（setter）。 如果没有设置器， 该值为undefined。(仅针对包含访问器或设置器的属性描述有效)

如下两个方法在添加属性时，都可以指定属性描述符
- `Object.create(proto, [ propertiesObject ])` 方法创建一个拥有指定原型和若干个指定属性的对象。
- `Object.defineProperties(obj, props)` 方法在一个对象上添加或修改一个或者多个自有属性，并返回该对象。

```
var obj = Object.create(Object.prototype, {
    a: {
        configurable: true,
        enumerable: true,
        value: 1,
        writable: true
    }
})

Object.defineProperties(obj, {
    b: {
        configurable: true,
        enumerable: true,
        value: 2,
        writable: true
    }
})
```
通过 `Object.getOwnPropertyDescriptor(obj, prop)` 方法可以返回指定对象上一个自有属性对应的属性描述符。（自有属性指的是直接赋予该对象的属性，不需要从原型链上进行查找的属性，也即是实例属性）
```
Object.getOwnPropertyDescriptor( obj , 'a' )
// 输出：
{
    configurable:true
    enumerable:true
    value:1
    writable:true
}
```

## 可枚举性 `enumerable`
可枚举性（`enumerable`）用来控制所描述的属性。具体来说，如果一个属性的 `enumerable` 为 `false`，下面三个操作不会取到该属性。

- `for…in` 循环
- `Object.keys` 方法
- `JSON.stringify` 方法

```
var obj = Object.create(Object.prototype, {
    a: {
        value: 1,
        enumerable: true
    },
    b: {
        value: 2,
        enumerable: false
    },
    c: {
        value: 3
        // enumerable 默认为 false
    }
})
// ① `for...in` 循环
for(var key in obj) {
    console.log(key)
}
// log: a

// ② `Object.keys` 方法
console.log(Object.keys(obj))
// log: ["a"]

// ③ `JSON.stringify` 方法
console.log(JSON.stringify(obj))
// log: {"a":1}
```
虽然 `enumerable` 为 `false` 的属性通过遍历操作无法获取到，但是我们还是可以直接获取到属性的值，例如 `obj.b`

## 原型属性和实例属性
javascript 中属性可以分为：实例属性和原型属性。原型属性是定义在对象的原型(`prototype`)中的属性，而实例属性一方面来自构造的函数中，或者是构造函数实例化后添加的新属性。

```
var o = {
    d: 4
}
var obj = Object.create(o, {
    a: {
        value: 1,
        enumerable: true
    },
    b: {
        value: 2,
        enumerable: true
    },
    c: {
        value: 3,
        enumerable: true
    }
})
```
如上代码中 `obj` 的就包含了一个原型属性 `d`，还有三个实例属性 `a, b, c` (都是可枚举的属性)。
我们注意到 `for...in` 循环和 `Object.keys` 方法都可以对对象进行遍历操作，那么二者是否有啥区别呢？执行以下如下代码：
```
for(key in obj) {
    console.log(key)
}
// log: a b c d

Object.keys(obj)
// log: ["a", "b", "c"]
```
我们可以注意到：`for...in` 循环可以遍历对象中所有可枚举的对象属性，包括原型属性和实例属性(也就是继承的属性和对象自有属性)；而 `Object.keys` 方法只能遍历出实例属性(也就是对象本身自有的属性)。

而通过 `hasOwnProperty` 方法就可以区分属性是否是原型属性还是实例属性。
```
obj.hasOwnProperty('a') // true
obj.hasOwnProperty('d') // false
```

而通过 `Object.getOwnPropertyNames(obj)` 方法返回一个由指定对象的所有自身属性的属性名（**包括不可枚举属性**）组成的数组，但不会获取原型链上的属性。
```
var o = {
    d: 4
}
var obj = Object.create(o, {
    a: {
        value: 1,
        enumerable: true
    },
    b: {
        value: 2,
        enumerable: false
    },
    c: {
        value: 3
        // enumerable 默认为 false
    }
})
Object.getOwnPropertyNames(obj) // ["a", "b", "c"]
```
----
补充一个知识点：ES6 class 类的内部所有定义的方法，都是不可枚举的（non-enumerable）；这一点和 ES5 有点差别（[详细内容参考](http://es6.ruanyifeng.com/#docs/class)）
```
class A{
    methoda() {}
    methodb() {}
}

function B() {

}
B.prototype.methoda = () => {};
B.prototype.methodb = () => {};


Object.keys(A.prototype) // []
Object.getOwnPropertyNames(A.prototype) // ["constructor", "methoda", "methodb"]

Object.keys(B.prototype) // ["methoda", "methodb"]
Object.getOwnPropertyNames(B.prototype) // ["constructor", "methoda", "methodb"]
```










