title: JS中是 pass by value 还是 pass by reference🤔
date: 2017-02-26 14:00:34
tags: [值传递, 引用传递]
categories: JS
---


在 JavaScript 中是传值(pass by value)还是传引用(pass by reference)呢？通常我们认为传参是原始类型（String, Number...）就是 pass by value，而引用类型（Array, Object）的话，就是 pass by reference。那么事实真的是如此吗？下面通过一些例子来了解一下。

<!-- more -->

## 案例
```
function changeStuff(a, b, c)
{
  a = a * 10;
  b.item = "changed";
  c = {item: "changed"};
}

var num = 10;
var obj1 = {item: "unchanged"};
var obj2 = {item: "unchanged"};

changeStuff(num, obj1, obj2);

console.log(num); // 10
console.log(obj1.item); // changed
console.log(obj2.item); // unchanged
```
假设：
- 如果 JS 中是 pass by value，那么例子中，修改 `b.item` 就不会导致函数外 `obj1` 发生变化；
- 如果 JS 中是 pass by reference，那么例子中，修改参数 `a` 也会生效，变量 `num` 就会变成 100；还有 `obj2.item` 也会变成 `changed`；

**那么 JS 中到底是传值还是传引用呢？答案是 pass by value，只不过对于引用类型，传递的 value 是其引用 reference。技术上，称之为 [call-by-sharing](https://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_sharing)**

那么对于这个例子，这么解释就通了，
1、对于参数 `a` 传递的是原始类型，单纯的传递 value，所以，函数 `changeStuff` 中即使对其进行了修改，也不会影响函数外的 `num`
2、对于参数 `b`, `c` 传递的是引用类型，那么就是传递了其引用，也即是 `b` 和 `obj1` 都指向了 `{item: "unchanged"}`，`c` 和 `obj2` 也是如此
- `b.item = "changed";` 是修改了其指向对象的的某一个属性，那么`b` 和 `obj1` 都同时指向的对象就发生变化，变成了 `{item: "changed"}`
- `c = {item: "changed"};` 是修改了参数 `c` 的指向，`c` 和 `obj2` 此时的指向不同，`c` 指向的是 `{item: "changed"}`，而 `obj2` 指向的是 `{item: "unchanged"}`

## 总结
>- **Javascript is always pass by value, but when a variable refers to an object (including arrays), the "value" is a reference to the object.**
- Changing the value of a variable never changes the underlying primitive or object, it just points the variable to a new primitive or object.
- However, changing a property of an object referenced by a variable does change the underlying object.

Example 1:
```
function f(a,b,c) {
    // Argument a is re-assigned to a new value.
    // The object or primitive referenced by the original a is unchanged.
    a = 3;
    // Calling b.push changes its properties - it adds
    // a new property b[b.length] with the value "foo".
    // So the object referenced by b has been changed.
    b.push("foo");
    // The "first" property of argument c has been changed.
    // So the object referenced by c has been changed (unless c is a primitive)
    c.first = false;
}

var x = 4;
var y = ["eeny", "miny", "mo"];
var z = {first: true};
f(x,y,z);
console.log(x, y, z.first); // 4, ["eeny", "miny", "mo", "foo"], false
```
Example 2:
```
var a = ["1", "2", {foo:"bar"}];
var b = a[1]; // b is now "2";
var c = a[2]; // c now references {foo:"bar"}
a[1] = "4";   // a is now ["1", "4", {foo:"bar"}]; b still has the value
              // it had at the time of assignment
a[2] = "5";   // a is now ["1", "4", "5"]; c still has the value
              // it had at the time of assignment, i.e. a reference to
              // the object {foo:"bar"}
console.log(b, c.foo); // "2" "bar"
```
## 其他
1、前段时间在微博上看到[阮一峰微博上说，函数参数默认值不是传值调用](http://weibo.com/1400854834/ErIrQ0BYg?type=comment#_rnd1488085633169)，例子如下：
```
let x = 99;
function foo(p = x + 1) {
  console.log(p)
}
foo(); // 100
x = 100;
foo(); // 101
```
那么是不是呢？在评论中，看到有一个比较合理的说法是：**其实就是传值调用**，只不过每执行一次foo()就传一次新值。传进去的值不会改变，除非再次调用foo()。例子如下：
```
let x = 99;
function foo(p = x + 1) {
  var count = 0;
  var intervalId = setInterval(() => {
    console.log(p);
    count++;
    if(count>10) clearInterval(intervalId);
  }, 1000)
}
foo();
setTimeout(() => {
  x = 100;
}, 3000)
```
可以发现，代码执行过程中，虽然外部 `x` 发生了变化，但是输出的值还一直是 100。

2、pass by value 和 pass by reference 的区别
![](http://cdn.objcer.com/pass-by-reference-vs-pass-by-value-animation.gif)

## 参考链接
[Javascript by reference vs. by value](http://stackoverflow.com/questions/6605640/javascript-by-reference-vs-by-value?answertab=votes#tab-top)
