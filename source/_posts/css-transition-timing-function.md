title: 关于 CSS transition-timing-function
date: 2017-03-18 16:48:37
tags: transition-timing-function
categories: CSS
---


在 CSS Animation 属性中可以设置和动画有关的参数，其中 transition-timing-function 是规定动画执行的速度曲线，常见的取值有两个：①步进函数 ②三次贝塞尔曲线
> Timing functions are either defined as a stepping function or a cubic Bézier curve.

[(W3C)The 'transition-timing-function' Property](https://www.w3.org/TR/2012/WD-css3-transitions-20120403/#transition-timing-function-property)
<!-- more -->

平时使用过程中，大多都是用 linear ease ease-in ease-out ease-in-out cubic-bezier(n,n,n,n) 这几个参数，这些方式过渡动画，会在每个关键帧之间插入补间动画，所以看起来动画的效果是平滑连贯的。

例如：
```
// html
<div class="box"></div>
// css
.box {
	position: absolute;
	left: 0px;
	width: 150px;
	height: 150px;
	background: gray;
	animation: move 2s linear forwards;
}
@keyframes move {
	0% {
		left: 0px;
	}
	100% {
		left: 600px;
	}
}
```
该例子中，div 节点就会匀速从左向右移动。

然而对于有些动画效果是不需要连贯的，只需要在关键帧之间跳跃，称之为逐帧动画；这个时候就应该使用 steps 方式过渡，可以指定的参数包括：`step-start | step-end | steps(<integer>[, [ start | end ] ]?)`

- `steps(<integer>[, [ start | end ] ]?)` 可以传入两个参数，第一个是大于0的整数，它是将间隔动画，也即是**两个关键帧**，等分成指定数目的小间隔动画；第二个参数，可以接受 start 和 end 两个值，指定在每个间隔的终点或是起点发生阶跃变化，默认为 end
- `step-start` 等同于 `steps(1,start)`：动画分成1步，动画执行时以 **间隔终点** 为起点开始变化；
- `step-end` 等同于 `steps(1,end)`：动画分成一步，动画执行时以 **间隔起点** 为起点开始变化。

![](http://7vikhl.com1.z0.glb.clouddn.com/step.png)

例如：
```
// html
<div class="box"></div>
// css
.box {
	position: absolute;
	left: 0px;
	width: 150px;
	height: 150px;
	background: gray;
	animation: move 2s steps(2, start) forwards;
}
@keyframes move {
	0% {
		left: 0px;
	}
	100% {
		left: 600px;
	}
}
```
该例子中，div 节点移动变化：`300px --> 600px`。如果修改成 `animation: move 2s steps(2, end) forwards;`，那么 div 节点移动变化：`0px --> 300px --> 600px`

逐帧动画例子可以参考: [css逐帧动画](http://www.jianshu.com/p/05c5a9b302d2)
