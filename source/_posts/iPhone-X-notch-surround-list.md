title: iPhone X 环绕刘海滚动列表
date: 2017-10-10 15:33:27
tags: DEMO
categories: JS
---

iPhone X 面市后，其异形屏给交互设计提供了更多想象的空间。在 Twitter 上，这位推友就针对刘海设计了列表环绕刘海滚动的效果。
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/4E124A83-5DD5-44C1-AA71-9394E34E8C57.png)

最近在 codepen 上看到已经有人实现了这个 demo：[https://codepen.io/davvidbaker/pen/KXgPyG](https://codepen.io/davvidbaker/pen/KXgPyG)，本文将图文结合分析一下实现这个效果的逻辑。

<!-- more -->

## DOM 结构
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/D75661B5-C160-4906-826D-E7908AA9B237.png)
```
<div class="outer">
    <div class="inner">
      <ul>
        <li>北京</li>
      </ul>
    </div>
    <div class="notch"></div>
  </div>
```
DOM 结构比较简单，主要包括列表 `ul`，其滚动容器为 `class="inner"` 的 DIV；刘海是 `class="notch"` 的 DIV。

## 滚动逻辑
为了实现列表环绕刘海滚动，需要在滚动事件中，计算处理每一行列表 X 轴方向的位置移动。
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/840F60D8-1D25-4B1C-BF50-C9F79DCC2AC9.png)

如上图列表向下滚动，上方的列表需要向右位移 `notch` 宽度；对应的，下方的列表需要向左位移 `notch` 宽度。我们需要对移动进行相应的计算处理，使位移线性变化，从而在滚动时，环绕效果自然。

### `distFromTop` 和 `distFromBottom`
列表在上下滚动过程中分为两部分：
- 向下滚动：上方列表向右移动进入刘海区域，下方列表向左移动离开刘海区域
- 向上滚动：上方列表向左移动离开刘海区域，下方列表向右移动进入刘海区域

这两部分刚好是相对应的，此处我们只分析**上方列表移动进入/离开刘海区域**，即可以了解滚动过程中的逻辑处理。首先介绍两个变量：`distFromTop` 和 `distFromBottom`

> `Element.getBoundingClientRect()` 方法返回元素的大小及其相对于视口的位置，返回值是一个 `DOMRect` 对象。`DOMRect` 对象包含了一组用于描述边框的只读属性 —— left、top、right 和 bottom，单位为像素。除了 width 和 height 外的属性都是相对于视口的左上角位置而言的，如下图：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/F772D60E-E17B-434F-A970-053BB77E53BB.png)

```
let notchRect = notch.getBoundingClientRect()
let itemRect = item.getBoundingClientRect()
let distFromTop = itemRect.bottom - notchRect.top
let distFromBottom = itemRect.top - notchRect.bottom
```

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/CB366BB3-C658-4514-96F7-103F5B712AD6.png)

这两个变量表征当前这一行列表距离上下刘海边界的位置，在滚动过程中根据 `distFromTop` 和 `distFromBottom` 变量触发不同的位移逻辑。

### 位移逻辑
分析**上方列表移动进入/离开刘海区域**时，列表位移的逻辑。
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/7B8867D1-1A5E-4DE6-A6CB-5B64209808F2.png)
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/E875669C-BEBE-4B6E-A883-EACF4CF29D63.png)

1️⃣ **阈值：`Threshold`**
在刘海边界上下 `Threshold` 区域定义为位移区域，列表进入这一区域即需要进行计算位移量，检测是否进入这一区域的逻辑判断为：
```
Math.abs(distFromTop) <= Threshold
```

2️⃣ **计算位移量**
考虑两个边界情况：
- 当 `distFromTop` 等于 `- Threshold`，即列表开始进入刘海区域
  此时位移量为 0
- 当 `distFromTop` 等于 `+ Threshold`，即列表完全进入刘海区域
  此时位移量为刘海宽度 `NotchWidth`

这个变换过程可以通过线性插值来完成，根据 `distFromTop` 变量计算在 Y 轴方向的比例关系，从而得到 X 轴方向的位移量。
> 关于线性插值参考：[维基百科：线性插值](https://zh.wikipedia.org/zh-cn/%E7%BA%BF%E6%80%A7%E6%8F%92%E5%80%BC)

在此处，我们可以简化理解，如下图
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/94A6750A-B927-4B11-8446-8F92DCFA7747.png)

计算公式如下：
```
let lerp = (v0, v1, t) => v0 + (v1 - v0) * t
```

在我们分析的这个情形：上方列表向右移动进入刘海区域，对应的调用为：
```
x = lerp(0, NotchWidth, (distFromTop + Threshold) / (Threshold * 2))
```


其他情况，位移量计算分别为：
- 在刘海区域内滚动
  判断条件：`distFromTop > 0 && distFromBottom < - Threshold`
  位移量始终为：刘海宽度 `NotchWidth`
- 下方列表移动进入/离开刘海区域
  判断条件：`Math.abs(distFromBottom) <= Threshold`
  位移量始终为：`x = lerp(NotchWidth, 0, (distFromBottom + Threshold) / (Threshold * 2))`
- 其他
  位移量始终为：0

## 完整代码
最终实现效果：[https://codepen.io/yingshandeng/pen/JrJRBR?editors=1010](https://codepen.io/yingshandeng/pen/JrJRBR?editors=1010)

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/iPhone%20X%20list.gif)

```
<html>

<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
  html,
  body {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 100%;
      height: 100%;
      padding: 0;
      margin: 0;
      background: #E3F3FD;
      font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  }
  .outer {
    position: relative;
    width: 640px;
    height: 310px;
    min-width: 640px;
    min-height: 310px;
    background-color: white;
    border-radius: 45px;
    -webkit-box-shadow: 0 0 20px 0 rgba(0, 0, 0, 0.10),
      inset 0 -5px 20px 0 rgba(0, 0, 0, 0.08);
    box-shadow: 0 0 20px 0 rgba(0, 0, 0, 0.10),
      inset 0 -5px 20px 0 rgba(0, 0, 0, 0.08);
  }
  .inner {
    position: absolute;
    width: 610px;
    height: 280px;
    margin: 15px;
    border-radius: 30px;
    background-color: #EBEBEB;
    overflow-x: hidden;
    overflow-y: scroll;
  }
  .notch {
    position: absolute;
    left: 15px;
    top: 79px;
    width: 22px;
    height: 152px;
    border-top-right-radius: 17px;
    border-bottom-right-radius: 17px;
    background-color: #fff;
  }
  ul {
    list-style: none;
    margin: 0 auto;
    padding-left: 10px;
  }
  li {
    padding: 6px 5px;
    border-bottom: 1px solid #dadada;
    transform-origin: center left;
  }

  *::-webkit-scrollbar {
    visibility: hidden;
  }
  </style>
</head>

<body>
  <div class="outer">
    <div class="inner">
      <ul>
        <li>北京</li>
        <li>天津</li>
        <li>河北</li>
        <li>山西</li>
        <li>内蒙古</li>
        <li>辽宁</li>
        <li>吉林</li>
        <li>黑龙江</li>
        <li>上海</li>
        <li>江苏</li>
        <li>浙江省</li>
        <li>安徽</li>
        <li>福建</li>
        <li>江西</li>
        <li>山东</li>
        <li>河南</li>
        <li>湖北</li>
        <li>湖南</li>
        <li>广东</li>
        <li>广西</li>
        <li>海南</li>
        <li>重庆</li>
        <li>四川</li>
        <li>贵州</li>
        <li>云南</li>
        <li>西藏</li>
        <li>陕西</li>
        <li>甘肃省</li>
        <li>青海</li>
        <li>宁夏</li>
        <li>新疆</li>
        <li>台湾</li>
        <li>香港特别行政区</li>
        <li>澳门</li>
      </ul>
    </div>
    <div class="notch"></div>
  </div>

  <script>
  let inner = document.querySelector('.inner')
  let items = document.querySelectorAll('li')
  let notch = document.querySelector('.notch')
  let notchRect = notch.getBoundingClientRect()

  inner.addEventListener('scroll', () => {
    window.requestAnimationFrame(scrollDection)
  })
  window.addEventListener('resize', () => {
    notchRect = notch.getBoundingClientRect()
  })

  const Threshold = 10
  const NotchWidth = 30
  let scrollDection = () => {
    let item
    for (item of items) {
      let itemRect = item.getBoundingClientRect()
      let distFromTop = itemRect.bottom - notchRect.top
      let distFromBottom = itemRect.top - notchRect.bottom
      let x

      if (Math.abs(distFromTop) <= Threshold) {
        x = lerp(0, NotchWidth, (distFromTop + Threshold) / (Threshold * 2))
      } else if (distFromTop > 0 && distFromBottom < - Threshold) {
        x = NotchWidth
      } else if (Math.abs(distFromBottom) <= Threshold) {
        x = lerp(NotchWidth, 0, (distFromBottom + Threshold) / (Threshold * 2))
      } else {
        x = 0
      }

      item.style.transform = `translateX(${x}px)`
    }
  }
  let lerp = (v0, v1, t) => v0 + (v1 - v0) * t

  scrollDection()
  </script>
</body>

</html>
```
