title: 实现一个 scroll-fixed-content 效果
tags:
---
🍉 scroll-fixed-content 效果如下（demo 中搜索框在向上滚动时，会固定在导航栏的位置）：

![](http://7vikhl.com1.z0.glb.clouddn.com/scroll-fixed-content.gif)

<!-- more -->

## 实现过程
分析一下 demo 中的结构：
- 在初始状态时（State One）时
	- header area：fixed 定位
	- top area：relative 定位，其中搜索框，absolute 定位
	- main area：relative 定位
- 在向上滚动（State Two）搜索框固定在导航栏位置时
	- header area：fixed 定位
	- top area：relative 定位，**其中搜索框，改成 fixed 定位，固定到导航栏位置**
	- main area：relative 定位

![](http://7vikhl.com1.z0.glb.clouddn.com/4E1CE12F-B7FF-47F1-8F4D-984EDA1DDE0A.png)

DOM 结构如下：
```
<header class="top-header">
    <span class="menu-icon">☰</span>
</header>

<div class="top">
    <div class="hero"></div>
    <div class="search">
        <input type="search" placeholder="Search or type URL" />
    </div>
</div>

<main>
    <div></div>
    <div></div>
    // ...
</main>
```
TODO ... 代码参考：。。。

为了实现这两个状态间的切换，最简单我们能想到的就是，监听 `scroll` 事件，根据页面 `scrollTop` 信息，通过添加、移除 class 的方式来实现。
```
window.addEventListener('scroll', () => {
    if (document.body.scrollTop > 170) {
        document.body.classList.add('fix-search');
    } else {
        document.body.classList.remove('fix-search');
    }
});
```


## 关于滚动事件

throttle。。。
```
```

## 参考链接
[Scroll-Then-Fix Content](https://css-tricks.com/scroll-fix-content/)
[]()