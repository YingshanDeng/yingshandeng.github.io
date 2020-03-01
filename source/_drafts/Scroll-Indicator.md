title: Scroll-Indicator
date: 2016-08-16 19:31:19
tags:
categories:
---

本文介绍两种方式实现页面滚动进度指示器。
![scroll-indicatior](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/C41FDF72-EDAE-49F4-A6FD-29EA6878C736.png)

<!-- more -->

### JS 方式

### 纯 CSS 方式


### CSS 渐变

> 详细的文档介绍请参考：[MDN参考文档](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Using_CSS_gradients)

CSS 渐变是 CSS3 中新增的 [image](https://developer.mozilla.org/zh-CN/docs/Web/CSS/image) 类型，CSS 渐变可以在两种颜色间制造出平滑的渐变效果。渐变包括两种类型：线性渐变 (linear)，通过 `linear-gradient` 函数定义，以及 径向渐变 (radial)，通过 `radial-gradient` 函数定义。

> **注**：一个 image CSS 数据类型可能像如下情形展示：
- 一个图像被引用为CSS <uri>数据类型使用url()方法；
- 一个CSS<gradient>;
- 页面的一个部分，定义在element()方法中；
```
url(test.jpg)                          url()方法, 只要test.jpg是图像文件
linear-gradient(to bottom, blue, red)  一个 <gradient>标签
element(#colonne3)                     页面的一部分, 使用了element()方法,如果 colonne3 是存在于页面中的一个元素id即可
```

1、简单的线性渐变

可以指定渐变方向，起始颜色和过渡颜色。例如：

```
/* 从上向下渐变，起始于红色，过渡到白色 */
background: linear-gradient(to bottom, red, white);
/* 从左上角向右下角渐变，起始于红色，过渡到白色 */
background: linear-gradient(to bottom right, red, white);
```

2、使用角度：可以设置特定的渐变角度
角度是指水平线与渐变线之间的角度，以逆时针方向旋转。总之，0deg 创建一个从底部到顶部的垂直渐变，当变成90deg时生成一个从左到右的水平渐变。

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/linear_red_angles.png)

3、色标
渐近线的颜色停止点在该位置有特定的颜色。该位置可以被指定为线长度的百分比或一个绝对长度。为实现期望的效果，可以指定任意多个颜色停止点。

如果指定位置使用百分比，那么 0% 表示起点，100%表示终点。然而，如果有需要，也可以使用范围之外的值。例如：

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/linear_colorstops1.png)
```
background: linear-gradient(to bottom, blue, white 80%, orange);
```

如果使用了多种颜色，但并没有指定位置时，默认是自动均匀的隔开。

4、通过线性渐变，生成一个三角形

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/triangle.png)

```
<html>

<head>
    <style>
    html,body {
        width: 100%;
        height: 100%;
        margin: 0;
    }
    div {
        width: 100%;
        height: 100%;
        background: linear-gradient(to right top, #0089f2 50%, #DDD 50%);
    }
    </style>
</head>

<body>
    <div></div>
</body>

</html>
```

解释一下：我们通过 linear-gradient 线性渐变从左下角向右上角渐变，两个渐变的颜色的位置都指定在 50%，这样就生成了一个三角形






先上代码：
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>target-input</title>
    <style>
    html,
    body {
        margin: 0;
    }

    body {
        background: linear-gradient(to right top, #0089f2 50%, #DDD 50%);
        background-size: 100% calc(100% - 100vh + 129px);
        background-repeat: no-repeat;
    }
    body:before {
        content: '';
        position: fixed;
        top: 128px;
        bottom: 0;
        width: 100%;
        z-index: -1;
        background: white;
    }

    main,
    header {
        padding: 10px 10%;
        box-sizing: border-box;
    }

    header {
        position: fixed;
        top: 0;
        height: 125px;
        width: 100%;
        background: white;
    }

    main {
        margin-top: 128px;
    }
    </style>
</head>

<body>
    <header>
        <h1>Scroll Indicator</h1>
    </header>
    <main>
        <!-- scroll content 滚动内容 -->
    </main>
</body>

</html>
```

