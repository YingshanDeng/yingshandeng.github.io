title: JS 数组排序 sort 方法
date: 2017-05-21 16:42:51
tags: Array.prototype.sort
categories: JS
---

*[问题引入]:*在推特上看到有人发了这样一段代码
```
var a = [0, -1, -2, 1];
var b = a.sort();
console.log(b);
```
输出的结果竟然是：`[-1, -2, 0, 1]`
显然该推友对 `Array.prototype.sort` 用法不熟悉导致的误解。🤓
<!-- more -->

## `Array.prototype.sort`
```
arr.sort()

arr.sort(compareFunction)
```
`sort` 方法对数组中的元素进行排序，返回数组；可以指定排序顺序比较函数，如果省略，那么默认排序顺序是根据字符串 Unicode 码点**升序排列**。
通过一个例子理解一下：
```
var scores = [1, 10, 21, 8];
scores.sort(); // [1, 10, 21, 8]
```
我们对数组 `scores` 进行排序，没有指明 `compareFunction` ，那么元素会按照转换为的字符串的诸个字符的 Unicode 码点进行升序排列。相当于是
```
var scores = ['1', '10', '21', '8'];
scores.sort();
```
> 关于码点，参考文章：[JavaScript Unicode 编码那些事](http://objcer.com/2017/05/21/JavaScript-Unicode/)

```
'1'.codePointAt(0) // 49
'8'.codePointAt(0) // 56
```
字符 `1` 的码点比字符 `8` 的码点小，所以 `10` 排在 `8` 前面。同理也就可以解释文章开头的例子了。

## `compareFunction`
比较函数格式如下：
```
function compare(a, b) {
    if (a is less than b by some ordering criterion) {
        return -1;
    }
    if (a is greater than b by the ordering criterion) {
        return 1;
    }
    // a must be equal to b
    return 0;
}
```
有点需要注意的是，比较函数必须返回 -1 或者 0 或者 1；而不能返回 true 或者 false。

为了实现元素为数字的数组按照数值升序排列，我们就需要传入一个比较函数 `compareFunction`，如下：
```
var scores = [1, 10, 21, 8];
scores.sort((a, b) => a - b) // 升序排列
scores.sort((a, b) => b - a) // 降序排列
```
通过比较函数，我们也可以对对象数组按照某个属性进行排序，很方便。

## sort 排序不一定是稳定的
> Quicksort is generally considered to be efficient and fast and so is used by V8 as the implementation for Array.prototype.sort() on arrays with more than 23 items. For less than 23 items, V8 uses insertion sort[2]. Merge sort is a competitor of quicksort as it is also efficient and fast but has the added benefit of being stable. This is why Mozilla and Safari use it for their implementation of Array.prototype.sort().

关于排序是否稳定，参考: [维基百科](http://imweb.io/topic/565cf7253ad940357eb99881)
这个图也能很好的理解
![](http://7vikhl.com1.z0.glb.clouddn.com/440px-Sorting_stability_playing_cards.svg.png)

## 参考链接
[MDN Array.prototype.sort](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)