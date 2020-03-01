title: 关于 CSS border-radius
date: 2017-03-17 21:46:43
tags: border-radius
categories: CSS
---


`border-radius` 用来设置边框圆角。当使用一个半径时确定一个圆形；当使用两个半径时确定一个椭圆，这个(椭)圆与边框的交集形成圆角效果。
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/FC08A931-6DE4-4CE0-A6F8-4F1B523E2007.png)

对于 `border-radius` 属性，我们最常见就是设置一个参数：
```
border-radius: 50px;
// 或者
border-radius: 50%;
```
而这种用法其实就是为四个角都指定了相同的圆角半径，然而对于 `border-radius` 其实有更加灵活的用法，可以分别设置四个角不同的圆角效果。

<!-- more -->

## 语法
`border-radius` 是一个简写属性，分别设置了：
- border-top-left-radius
- border-top-right-radius
- border-bottom-right-radius
- border-bottom-left-radius

1、`border-radius` 可以分别指定1，2，3，4个值，值的类型可以是 length 或者 percentage。
- 指定一个值：四个角都设定相同的圆角值
- 指定两个值：参数一指定 top-left 和 bottom-right；参数二指定 top-right 和 bottom-left
- 指定三个值：参数一指定 top-left；参数二指定 top-right 和 bottom-left；参数三指定 bottom-right
- 指定四个值：四个参数分别指定 top-left，top-right，bottom-right，bottom-left（顺时针方向）

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/33F068D3-6BC7-4EE1-94BD-1B83632382FA.png)

2、`border-radius` 可以为圆角指定两个半径，上述我们谈到的都是指定了一个圆角半径，那么就是一个正圆形；而指定两个半径，那么就是一个椭圆形。用法是通过一个斜线分割开，斜线左方是 x 轴方向的半径，右方是 y 轴方向的半径。当然，斜线左右两侧可以分别指定 1，2，3，4 个参数，所以有较多的组合。
例如：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/D6EBFB3B-41B5-4249-84C6-5F1773F8C480.png)

## 参考链接
[MDN border-radius](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius)

