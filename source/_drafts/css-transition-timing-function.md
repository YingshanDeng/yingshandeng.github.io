title: 关于 CSS 逐帧动画
tags:
---

## transition-timing-function
在 CSS Animation 属性中可以设置和动画有关的参数，其中 transition-timing-function 是规定动画执行的速度曲线，常见的取值有两个：①步进函数 ②三次贝塞尔曲线
> Timing functions are either defined as a stepping function or a cubic Bézier curve.

其中1-7大多都有介绍，但是animation-timing-function是控制时间的属性

在取值中除了常用到的 三次贝塞尔曲线 以外，还有个让人比较困惑的 steps() 函数

animation默认以ease方式过渡，它会在每个关键帧之间插入补间动画，所以动画效果是连贯性的

除了ease，linear、cubic-bezier之类的过渡函数都会为其插入补间。但有些效果不需要补间，只需要关键帧之间的跳跃，这时应该使用steps过渡方式



animation-timing-function 规定动画的速度曲线

## 逐帧动画



## 参考链接：
[]()
[]()



https://www.w3ctrain.com/2016/04/06/pure-css-pie-animation/

http://www.cnblogs.com/aaronjs/p/4642015.html

http://www.jianshu.com/p/05c5a9b302d2
