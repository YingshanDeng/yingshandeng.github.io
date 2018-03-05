title: 检测鼠标指针移动离开页面
date: 2018-02-26 23:21:40
tags: relatedTarget
categories: JS
---

![](http://7vikhl.com1.z0.glb.clouddn.com/banner-purple.jpg)
本文将介绍一种方法用于检测鼠标指针在页面中移动时，判断是否离开页面 👉

<!-- more -->

## `relatedTarget`
W3C在 `mouseover` 和 `mouseout` 事件中添加了 `relatedTarget` 属性。`relatedTarget` 属性返回与事件的目标节点相关的节点元素。
- 对于 `mouseover` 事件来说，该属性表示来自哪个元素
- 对于 `mouseout` 事件来说，该属性表示去往的那个元素

对于其他类型的事件来说，这个属性没有用。
而Microsoft添加了两个属性：
- `fromElement` 在 `mouseover` 事件中表示鼠标来自哪个元素
- `toElement` 在 `mouseout` 事件中指向鼠标去往的那个元素

跨浏览器脚本：
- 如果你想知道鼠标来自哪个元素(`mouseover` 事件)：
```
function doSomething(e) {
  if (!e) var e = window.event;
  var relTarg = e.relatedTarget || e.fromElement;
}
```

- 如果你想知道鼠标去往哪个元素在(`mouseout` 事件)：
```
function doSomething(e) {
  if (!e) var e = window.event;
  var relTarg = e.relatedTarget || e.toElement;
}
```

## `mouseover`, `mouseout`, `mouseenter`, `mouseleave`
关于鼠标移动，有两组容易混淆的事件：
- `mouseover`, `mouseout`: **事件会冒泡**
- `mouseenter`, `mouseleave`: 不会冒泡

如下例子：
![](http://7vikhl.com1.z0.glb.clouddn.com/E26BF2A3-9639-4EC5-B23F-7F3D8FB3BB31.png)
- 当鼠标移动进入 outer 元素，会触发 outer 元素的 `mouseover`, `mouseenter` 事件；
- 当鼠标从 outer 元素移动进入 inner 元素，会触发 inner 元素的 `mouseover`, `mouseenter` 事件；**同时会触发 outer 元素的 `mouseover` 事件**

## 检测光标移动离开页面
通过 `mouseout` 鼠标事件，判断鼠标去往哪个元素
```
document.body.addEventListener('mouseout', (evt) => {
  if (!evt) var evt = window.event
  var to = evt.relatedTarget || evt.toElement
  if (!to || to.nodeName == "HTML") {
    alert("left window");
  }
})
```

检测鼠标指针移动离开页面的应用的例子：拖拽一个DIV节点，在鼠标移动离开页面后，应该停止拖拽
```
<html>
<head>
  <meta charset="UTF-8">
  <title>拖拽实现</title>
  <style>
    .block {
      position: absolute;
      left: 100px;
      top: 100px;
      height: 100px;
      width: 40px;
      height: 40px;
      background: red;
    }
  </style>
</head>
<body>
  <div class="block"></div>
  <script>
    var blcokEle = document.querySelector(".block")
    var blockCurrPos = {x: -1, y: -1} // block element current position
    var mouseStartPos = {x: -1, y: -1} // mouse start position
    var canDrag = false
    blcokEle.addEventListener('mousedown', (evt) => {
      canDrag = true
      let rect = blcokEle.getBoundingClientRect()
      Object.assign(blockCurrPos, {x: rect.left, y: rect.top})
    })
    blcokEle.addEventListener('mouseup', (evt) => {
      canDrag = false
    })

    document.body.addEventListener('mousedown', (evt) => {
      if (!canDrag) { return }
      Object.assign(mouseStartPos, {x: evt.clientX, y: evt.clientY})
    })
    document.body.addEventListener('mousemove', (evt) => {
      if (!canDrag) { return }
      var deltaX = evt.clientX - mouseStartPos.x
      var deltaY = evt.clientY - mouseStartPos.y

      blockCurrPos.x += deltaX
      blockCurrPos.y += deltaY

      mouseStartPos = {
        x: evt.clientX,
        y: evt.clientY
      }
      blcokEle.style['left'] = `${blockCurrPos.x}px`
      blcokEle.style['top'] = `${blockCurrPos.y}px`
    })
    document.body.addEventListener('mouseout', (evt) => {
      if (!canDrag) { return }
      var to = evt.relatedTarget || evt.toElement;
      if (!to || to.nodeName == "HTML") {
          // stop your drag event here
          canDrag = false
      }
    })
  </script>
</body>
</html>
```

## 参考链接
[How can I detect when the mouse leaves the window?
](https://stackoverflow.com/questions/923299/how-can-i-detect-when-the-mouse-leaves-the-window/26332723)
