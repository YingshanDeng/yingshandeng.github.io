title: javascript this keyword
tags:
---

<!-- more -->

##

箭头函数在设计中使用的是 Lexical this，即**这个函数被创建时的所在环境的 this 就是函数内部的 this**
```
function printThis() {
    let print = () => console.log(this);
    print();
}
printThis.call([1]); // [1]
```

```
function printThis() {
    let print = function() {
        console.log(this)
    }
    print();
}
printThis.call([1]); // window
```


##


this的查找this的查找是很多人迷茫的一点，也似乎有很多人抱有this不稳定这样的看法，实在令人无语。this的查找可以说是3种对象查找中最为简单的，因为其实this对象的确定根本没有一个“查找”的过程。

首先，this对象只会在一个函数中需要确定，如果是在全局域下，this永远为Global对象，在浏览器中通常就是window对象。

而在javascript中，函数的调用一共有4种方式：

Function Invocation Pattern 诸如`foo()`的调用形式被称为Function Invocation Pattern，是函数最直接的使用形式，注意这里的foo是作为单独的变量出现，而不是属性。在这种模式下，foo函数体中的this永远为Global对象，在浏览器中就是window对象。

Method Invocation Pattern诸如`foo.bar()`的调用形式被称为Method Invocation Pattern，注意其特点是被调用的函数作为一个对象的属性出现，必然会有“.”或者“[]”这样的关键符号。在这种模式下，bar函数体中的this永远为“.”或“[”前的那个对象，如上例中就一定是foo对象。

Constructor Pattern`new foo()`这种形式的调用被称为Constructor Pattern，其关键字`new`就很能说明问题，非常容易识别。在这种模式下，foo函数内部的this永远是new foo()返回的对象。

Apply Pattern`foo.call(thisObject)`和`foo.apply(thisObject)`的形式被称为Apply Pattern，使用了内置的`call`和`apply`函数。在这种模式下，`call`和`apply`的第一个参数就是foo函数体内的this，如果thisObject是`null`或`undefined`，那么会变成Global对象。

应用以上4种方式，确定一个函数是使用什么样的Pattern进行调用的，就能很容易确定this是什么。另外，this是永远不会延作用域链或原型链出现一个“查找”的过程的，只会在函数调用时就完全确认。



http://www.quirksmode.org/js/this.html

http://mao.li/javascript/javascript-this/

```
```

##
```
```