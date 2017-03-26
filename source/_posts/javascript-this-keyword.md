title: 详解 javascript this 关键字
date: 2017-03-26 17:02:09
tags: this
categories: JS
---


`this` 是 Javascript 语言的一个关键字。 它代表函数运行时，自动生成的一个内部对象，只能在函数内部使用。而这个内部对象 this 指向谁有时却让我们很困惑。在绝大多数情况下，函数的调用方式决定了 this 的值。this 不能在执行期间被赋值，在每次函数被调用时 this 的值也可能会不同。

<!-- more -->
## this 指向谁

### Function Invocation Pattern
诸如 `foo()` 的调用形式被称为 Function Invocation Pattern，是函数最直接的使用形式，注意这里的 foo 是作为单独的变量出现，而不是属性。在这种模式下，foo 函数体中的 this 永远为 Global 对象，在浏览器中就是 window 对象。
```
// 例子1：
var a = 1;
function foo() {
    console.log(this.a, this === window)
}
foo() // 1 true

// 例子2：
var a = 1;
function foo() {
    console.log('foo: ', this.a, this === window)
    function bar() {
        console.log('bar: ', this.a, this === window)
    }
    bar() // bar:  1 true
}
foo() // foo:  1 true
```
> 需要注意的是：在严格模式('use strict')中的默认的 this 不再是 window，而是 undefined.

### Method Invocation Pattern
诸如 `foo.bar()` 的调用形式被称为 Method Invocation Pattern，注意其特点是被调用的函数作为一个对象的属性出现，必然会有“.”或者“[]”这样的关键符号。在这种模式下，bar 函数体中的this永远为“.”或“[”前的那个对象，如下面例中就 `foo` 函数中 `this` 就是指向 `obj` 对象。
```
var obj = {
    a: 1,
    foo: function() {
        console.log(this.a, this === obj) // 1 true
    }
}

obj.foo();
```

### Constructor Pattern
`new foo()` 这种形式的调用被称为 Constructor Pattern，其关键字 `new` 就很能说明问题，当一个函数被作为一个构造函数来使用（使用new关键字），它的 `this` 与即将被创建的新对象绑定。如下面例子中，`foo` 函数内部的 `this` 永远是 `new foo()` 返回的对象。
```
function foo() {
    this.a = 1;
}

var f = new foo();
console.log(f.a) // 1
```

### Apply Pattern
`foo.call(thisObject)` 和 `foo.apply(thisObject)` 的形式被称为 Apply Pattern，使用了内置的 `call` 和 `apply` 函数。在这种模式下，`call` 和 `apply` 的第一个参数就是 `foo` 函数体内的 `this`，如果 `thisObject` 是 `null` 或 `undefined`，那么会变成 Global 对象。
```
function add(c, d) {
    return this.a + this.b + c + d;
}
var obj = {
    a: 1,
    b: 2
}
add.call(obj, 3) // 6
add.apply(obj, [3, 4]) // 10
```

>this 对象只会在一个函数中需要确定，如果是在全局域下，this 永远为 Global 对象，在浏览器中通常就是 window 对象。
应用以上4种方式，确定一个函数是使用什么样的 Pattern 进行调用的，就能很容易确定 this 是什么。另外，this 是永远不会延作用域链或原型链出现一个“查找”的过程的，只会在函数调用时就可以完全确认。

### 常见例子
```
var x = 10;
var obj = {
    x: 20,
    f: function() {
        console.log(this.x);
        var foo = function() {
            console.log(this.x);
        }
        foo(); // (2)
    }
};

obj.f(); // (1)
```
其中两处调用(1)、(2) 分别输出： 20、10；
- (1): `obj.f()` 调用，被调用的函数 `f` 作为 `obj` 对象的属性出现，属于 Method Invocation Pattern，`f` 函数内的 `this` 指向 `obj`, 所以 `f` 函数中输出 `this.x` 为20
- (2): `f` 函数中调用 `foo` 函数时是作为单独的变量出现，而非某个对象的属性，属于 Function Invocation Pattern，所以其中 `this` 指向全局变量 window, 输出 `this.x` 时找到全局变量中 `x` 的值 10，所以输出为 10

那么如果也希望(2)处调用输出 20，要如何修改呢？👇
```
var x = 10;
var obj = {
    x: 20,
    f: function() {
        console.log(this.x);
        var that = this;
        var foo = function() {
            console.log(that.x);
        }
        foo(); // (2)
    }
};

obj.f(); // (1)
```

## bind 函数和箭头函数

### bind 函数
ES5 引入了 `bind` 方法来设置函数的 `this` 值，而不用考虑函数如何被调用的。调用 `f.bind(someObject)` 会创建一个与 f 具有相同函数体和作用域的函数，但是在这个新函数中，this 将永久地被绑定到了 bind 的第一个参数，无论这个函数是如何被调用的。
```
function foo() {
    return this.a;
}

var bar = foo.bind({
    a: 'hello'
});
console.log(bar()) // hello


var o = {
    a: 'world',
    foo: foo,
    bar: bar
};
console.log(o.foo(), o.bar()) // world hello
```

### 箭头函数
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


## DOM事件处理函数中的 this
为 DOM 元素添加事件处理函数大致有以下两种方式：

### 方式一
`element.onclick = doSomething;`，函数被用作事件处理函数时，它的this指向触发事件的元素
```
<button id='btn1'>click me! btn1</button>
<button id='btn2'>click me! btn2</button>
<script>
function clickHandler() {
	console.log(this.id)
}
var btn1Element = document.getElementById('btn1'),
	btn2Element = document.getElementById('btn2');
btn1Element.onclick = clickHandler;
btn2Element.onclick = clickHandler;
</script>
```
在这个例子中，点击 button 之后，会执行其 `onclick` 事件回调函数 `clickHandler`，执行时，函数中的 `this` 都会指向触发事件的 button 元素（`onclick` 是 button 元素的一个属性）

### 方式二
`<element onclick="doSomething()">`，在内联事件处理函数中的 `this`，并不指向触发事件的元素
```
<button id='btn1' onclick="clickHandler()">click me! btn1</button>
<button id='btn2' onclick="clickHandler()">click me! btn2</button>
<script>
function clickHandler() {
	console.log(this, this.id)
}
</script>
```
在这个例子中，点击 button 之后，会执行 `clickHandler()` 函数，这属于 Function Invocation Pattern，所以此时函数中的 `this` 指向为 windows（严格模式下为 undefined）

如果希望在内联事件处理函数中的 `this` 也指向该元素，那么我们可以用 `bind` 函数指定其 `this` 值。
>当然也可以使用 `call` 或者 `apply` 方法。[JavaScript 中的 apply、call、bind](http://objcer.com/2017/02/19/apply-call-bind/)


```
<button id='btn1' onclick="clickHandler.bind(this)()">click me! btn1</button>
<button id='btn1' onclick="clickHandler.call(this)">click me! btn1</button>
<button id='btn1' onclick="clickHandler.apply(this)">click me! btn1</button>
```

### 区别
通过以上两种方式为 DOM 元素添加事件处理函数后，我们可以输出看看：
```
var btn1Element = document.getElementById('btn1');
console.log(btn1Element.onclick)
```
- 方式一输出：
```
function clickHandler() {
	console.log(this.id)
}
```
- 方式二输出：
```
function onclick(event) {
  clickHandler()
}
```

通过以上也很好说明了，前者中的 `this` 指向触发事件的 DOM 元素；而后者中的 `this` 指向全局对象 window

### 总结
通过以上的了解，不同方式为 DOM 元素添加事件处理函数，函数中的 `this` 指向可能不同，需要特别留意。

通过以下方式添加的事件处理函数，`this` 都正常指向 DOM 元素：
```
element.onclick = doSomething
element.addEventListener('click',doSomething, false)
element.onclick = function () { this.style.color = '#cc0000'; }
<element onclick="this.style.color = '#cc0000';">
<element onclick="doSomething.bind(this)()">
<element onclick="doSomething.call(this)">
<element onclick="doSomething.apply(this)">
```

而通过以下方式添加的事件处理函数，`this` 并不指向 DOM 元素：
```
element.onclick = function () { doSomething() }
element.attachEvent('onclick', doSomething)
<element onclick="doSomething()">
```

## 参考链接
[MDN this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)
[The this keyword](http://www.quirksmode.org/js/this.html)

