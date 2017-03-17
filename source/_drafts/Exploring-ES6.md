title: Exploring ES6
tags:
---

本文将对 ES6 中新增内容进行学习整理🤓

> 推荐一本介绍 ES6 的在线书籍：[Exploring ES6](http://exploringjs.com/es6/index.html)
中文翻译：[Exploring ES6中文](http://es6-org.github.io/exploring-es6/)

<!-- more -->

## Symbols
Symbols 是 ECMAScript 6 中新增的一个原始类型(primitive type)，通过一个工厂方法(factory function)创建 `symbol` :
```
// 语法
Symbol([description])

const mySymbol = Symbol('mySymbol');
```
每次调用该工厂方法，都会创建一个新的，独一无二的 `symbol`
```
Symbol('foo') === Symbol('foo'); // false
```
其中可选参数 `description` 为描述，在调试时打印输出。
```
> mySymbol
Symbol(mySymbol)
```

*参考链接*
[Symbols](http://exploringjs.com/es6/ch_symbols.html)

## Set

## Map WeakMap

## Proxy

## Reflection
