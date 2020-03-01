title: Unicode 之神奇的空格
date: 2017-05-22 21:37:42
tags: space
categories: Unicode
---


![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/IMG_3319.PNG)

<!-- more -->

## 问题引入
```
var a = "<p>The app&nbsp;</p>";
var b = "<p>The app </p>"
```
有如上两个两个字符串（其中前者是从 WYSIWYG 编辑器中获取到的字符串数据）， 我们已知两个字符串表示的内容是一样的，我们如何通过代码来判断他们相等呢。

看到这个问题，很简单的嘛 🤣 `&nbsp;` 是空格的转义字符，那我们可以先把 `&nbsp;` 替换成空格
```
function replaceNbsps(str) {
    return str.replace(/&nbsp;/g, ' ');
}
console.log(replaceNbsps(a)) // "<p>The app </p>"
replaceNbsps(a) == b // false
```
替换后，两个字符串看起来一模一样了，但是执行结果却是 `false` 🤔 这是为啥呢？

其实字符串 `b` 中暗含玄机：
```
var b = "<p>The app </p>"
var b = "<p>The app\u00a0</p>"
```
这其中的包含的一个空格字符，其 Unicode 为 `U+00A0`，十进制值为：160。莫非这两个空格字符不一样？

没错❗通过正则替换的空格字符的 Unicode 为 `U+0020`，十进制值为：32

Unicode | 十进制值 | 描述 | 拉丁字母
 ---- | ---- | ----
U+0020 | 160 | [空格](https://zh.wikipedia.org/zh-cn/%E7%A9%BA%E6%A0%BC) | 基本拉丁字母
U+00A0 | 32 | [不换行空格](https://zh.wikipedia.org/zh-cn/%E4%B8%8D%E6%8D%A2%E8%A1%8C%E7%A9%BA%E6%A0%BC) | 拉丁字母-1 辅助

## 空格 `U+0020`
我们通过键盘点击空格键输入的空格，就是此类空格，普通半角空格，显示为空白字符。
```
var space1 = '\u0020';
var space2 = ' ';
space1 == space2 // true

space1.charCodeAt(0) // 32
```

## 空格 `U+00A0`
这类空格字符称之为**不换行空格**。用途是禁止换行。

在 HTML 网页显示中，会自动合并多个连续的空白字符；但是如果使用此类字符，就会禁止合并，因此这类字符也称之为*硬空格*。在网页显示中其转义字符为 `&nbsp;`
```
var dom = document.querySelector('div'); // DOM 元素

// ① 自动合并空格
dom.innerText = 'a  b';
dom.innerText = 'a\u0020\u0020b'
dom.innerText.length // 3

// ② 禁止合并
dom.innerText = 'a\u00a0\u00a0b'
// 或者使用空格的转义字符 &nbsp; ，此时需要用 innerHTML
$0.innerHTML = 'a&nbsp;&nbsp;b'
dom.innerText.length // 4
```
对于 ① 我们在页面中会观察到，DOM 元素只会展示一个空格（两个空格合并成一个）；而对于 ② 就会显示两个空格。

## 解决问题
了解了原因，现在就可以解决第一节中的问题了。
```
function replaceNbsps(str) {
    return str.replace(/&nbsp;|\u00a0/g, ' ');
}

replaceNbsps(a) == replaceNbsps(b) // true
```

我们只需要统一把空格都转换成 Unicode 为 `\u0020` 的空格，然后再进行比较就可以了。


## 参考链接
[空格](https://zh.wikipedia.org/zh-cn/%E7%A9%BA%E6%A0%BC)
[不换行空格](https://zh.wikipedia.org/zh-cn/%E4%B8%8D%E6%8D%A2%E8%A1%8C%E7%A9%BA%E6%A0%BC)
