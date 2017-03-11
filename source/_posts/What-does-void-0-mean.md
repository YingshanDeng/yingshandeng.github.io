title: '关于 ''void 0'' 和 ''javascript:void(0)''❓'
date: 2017-03-12 00:55:34
tags: ['void 0', 'javascript:void(0)']
categories: JS
---

或许我们都在代码中见到过 `void 0` 和 `javascript:void(0)`，这两个到底是什么含义呢？
<!-- more -->

## `void 0`
`void` 运算符会对给定的表达式进行求值，然后直接返回 undefined。
*语法：*
```
void expression
```
*例如：*
```
void 0
void (0)
void "hello"
void (new Date())
//all will return undefined
```
这似乎看起来没有什么用处，因为如果我们需要返回 `undefined`，那我们直接使用 `undefined` 就好了嘛。
> The global undefined property represents the primitive value undefined. It is one of JavaScript's primitive types.

没错，对于正常情况下，直接使用 `undefined` 比 `void 0` 更加简单而且浅显易懂；但是在 JavaScript 中有一点比较特殊：`undefined` 并不是保留关键字，它只是一个全局属性，也就是说 `undefined` 只是一个变量名而已，我们也可以对它进行赋值：
```
alert(window.hasOwnProperty('undefined')); // alerts "true"

alert(undefined); //alerts "undefined"
var undefined = "new value";
alert(undefined) // alerts "new value"
```
注意：这种情况已经得到改善了，现代浏览器中，`undefined` 作为全局属性只是可读（read-only），但是在局部作用域中，还是可以对其进行再赋值：
```
function test() {
    var undefined = 'hello';
    console.log('---', undefined) // --- hello
}
test();
```

由于 `undefined` 值可能并不是 `undefined` 的问题，所以 `void 0` 就凸显作用了，`void 0` 使用返回 `undefined`

那么为什么会是 `void 0` 呢？能不能是 `void 100`，`void 200`，`void "hello"` 这些呢？当然也是可以的，如果你喜欢的话，使用 `void 0` 的好处是 0 简短且符合编程语言习惯。



## `javascript:void(0)`
在 `a` 标签（超链接）中，我们可能经常会遇到这种情况：
```
<a href="javascript:void(0);" >test</a>
```
当用户点击一个以 javascript: URI 时，浏览器会对冒号后面的代码进行求值，例如：
```
<a href="javascript:void(document.body.style.backgroundColor='green');">
  点击这个链接会让页面背景变成绿色。
</a>
```
而 `void(0)` 始终返回  `undefined`，所以 `<a href="javascript:void(0);" >test</a>` 就创建了一个死超链接，点击后，不会发生任何事情。

**注意**，虽然这么做是可行的，但利用 javascript: 伪协议来执行 JavaScript 代码是不推荐的。

如果想要一个链接点击后不做任何事情，或者响应点击而完成其他事情，可以设置其属性 href = "#"，然后绑定 `onclick` 事件，这样会有一个问题，就是当页面有滚动条时，点击后会返回到页面顶端，用户体验不好；当然有解决方法，我们可以在 `onclick` 事件中返回 `false` 即可。
```
<a href="#" onclick="return false;">test</a>
// 等价于
<a href="javascript:void(0);" >test</a>
```

`href="javascript:void(0)"` 与 `href="#"` 的区别：
- `javascript:void(0)` 仅仅表示一个死链接。
- `#` 包含了一个位置信息，默认的锚是 `#top`，也就是网页的上端。在页面很长的时候会使用 `#` 来定位页面的具体位置，格式为：`# + id`。

## 参考链接
[What does 'void 0' mean?](http://stackoverflow.com/questions/7452341/what-does-void-0-mean)
[What does 'javascript:void(0)' mean?](http://stackoverflow.com/questions/1291942/what-does-javascriptvoid0-mean)
