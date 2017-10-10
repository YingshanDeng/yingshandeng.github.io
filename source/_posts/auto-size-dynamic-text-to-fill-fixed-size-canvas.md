title: 动态文本自适应绘制到指定大小的 Canvas
date: 2017-10-10 15:15:11
tags: Canvas
categories: JS
---

![](http://7vikhl.com1.z0.glb.clouddn.com/1-UGtq-cSfbuekV8aHalVIAw.png)
本文将探究动态文本如何自适应绘制到指定大小的 Canvas 🎏

<!-- more -->

## `CanvasRenderingContext2D.fillText()`
```
void ctx.fillText(text, x, y [, maxWidth]);
```
在 (x, y) 位置填充文本。如果第四个参数提供了最大宽度，文本会进行缩放以适应最大宽度。如果绘制的文本实际宽度超过 `maxWidth`，那么会在水平方向上进行缩放，文字可能被压缩变形。
![](http://7vikhl.com1.z0.glb.clouddn.com/fillText.png)

## 限制宽度
```
ctx.measureText(text);
```
传入要绘制的文本内容，返回绘制到当前 Canvas 时文本的宽度。我们可以设置一个较大的字体，然后递减字体大小，直到绘制的文本宽度满足 Canvas 宽度限制，由此得到文本绘制到 Canvas 中合适的字体大小
```
function dynamicFitTextOnCanvas (text, fontface, desiredWidth) {
  var canvas = document.createElement('canvas')
  var context = canvas.getContext('2d')
  // start with a large font size
  var fontsize = 500
  do {
    context.font = `${fontsize}px ${fontface}`
    fontsize --
  } while (context.measureText(text).width > desiredWidth)
  return fontsize
}
```

显然字体大小 fontsize 逐次递减一，效率不够好，我们可以通过二分递减来优化这个过程
```
function dynamicFitTextOnCanvas (text, fontface, desiredWidth) {
  var canvas = document.createElement('canvas')
  var context = canvas.getContext('2d')
  // start with a large font size
  var fontsize = 500
  return measureTextBinary(context, text, 0, fontsize, fontface, desiredWidth)
}
function measureTextBinary (context, text, min, max, fontface, desiredWidth) {
  if (max - min < 1) {
    return Math.floor(min)
  }
  var cur = min + (max - min) / 2
  context.font = `${cur}px ${fontface}`
  var measureWidth = context.measureText(text).width
  if (measureWidth > desiredWidth) {
      return measureTextBinary(context, text, min, cur, fontface, desiredWidth)
  } else {
      return measureTextBinary(context, text, cur, max, fontface, desiredWidth)
  }
}
```

## 限制宽度和高度
注意到 `measureText` 方法只能测量绘制文本的宽度，无法得到高度信息；我们可以往 document 中添加一个辅助的 `div` 节点，设置其 `innerText` 为要绘制的文本，再设置其字体大小，通过 `getBoundingClientRect` 方法或者 `offsetWidth/offsetHeight` 属性就可以得到绘制指定字体大小的文本宽高；同理，通过递减字体大小，使得绘制的文本宽高满足 Canvas 的宽高限制；由于有了前面的经验，我们也用二分递减提高执行效率

```
function dynamicFitTextOnCanvas (text, fontface, desiredWidth, desiredHeight) {
  var tmpDiv = document.querySelector('.tmp-div')
  if (!tmpDiv) {
    tmpDiv = document.createElement('div')
    tmpDiv.classList.add('tmp-div')
    tmpDiv.style['position'] = 'absolute'
    tmpDiv.style['left'] = '-100000px'
    tmpDiv.style['top'] = '-100000px'
    tmpDiv.style['visibility'] = 'hidden'
    document.body.appendChild(tmpDiv)
    tmpDiv.innerText = text
  }
  // start with a large font size
  var fontsize = 500
  return measureDivBinary(tmpDiv, 0, fontsize, fontface, desiredWidth, desiredHeight)
}
function measureDivBinary (tmpDiv, min, max, fontface, desiredWidth, desiredHeight) {
  if (max - min < 1) {
    return Math.floor(min)
  }
  var cur = min + (max - min) / 2
  tmpDiv.style['font'] = `${cur}px ${fontface}`
  tmpDiv.style['line-height'] = `${cur}px`
  var measureRect = tmpDiv.getBoundingClientRect()
  if (measureRect.width > desiredWidth || measureRect.height > desiredHeight) {
    return measureDivBinary(tmpDiv, min, cur, fontface, desiredWidth, desiredHeight)
  } else {
    return measureDivBinary(tmpDiv, cur, max, fontface, desiredWidth, desiredHeight)
  }
}
```

### 关于 `line-height`
![](http://7vikhl.com1.z0.glb.clouddn.com/2587037021-55bacfa692fb8.png)
通过上图来理解行高，行距
- **行高** (line-height) 是指文本行基线间的垂直距离，上图中两条红线之间的距离就是行高
- 上一行的底线和下一行的顶线之间的距离就是**行距**，而同一行顶线和底线之间的距离是 font-size 的大小
- 行距的一半是半行距: 半行距 = (line-height - font-size) / 2，当 line-height < font-size，半行距为负值，这时候两行之间就会重叠

我们发现，font-size 大小即为文字绘制后的实际高度，所以在上面代码中，我们对辅助 `div` 节点也设置了 `line-height = font-size`，使得到的文字高度更加精确（去掉了两个半行距）

### 存在的问题
1️⃣ 最小字体大小限制
在 Chrome 浏览器中，设定 CSS `font-size` 小于 12px ，依然会以 12px 进行展示，也就是说 Chrome 浏览器允许设置的最小 `font-size` 为 12px（Safari 不存在这个问题）。所以如果限制绘制的 Canvas 的宽高小于辅助 `div` 节点设置 `font-size` 为 12px 的宽高，那么无法通过计算得到合适绘制的 `font-size`

**为了解决这个问题：**只能通过设置一个较大的 `font-size`，在 Canvas 上进行绘制，然后对这个 Canvas 缩小到限制的宽高

2️⃣ 高清屏绘制文字模糊
关于这个问题，解决方案参考：[高清屏中 Canvas 的绘制](https://objcer.com/2017/10/10/High-DPI-Canvas-Render/)
