title: 实现一个字数限制的输入框
date: 2017-10-10 15:24:25
tags: DEMO
categories: JS
---

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/twitter-input-box.png)
在 Twitter 发推输入框中，有一个字数限制的逻辑，超出字数限制部分的文字会设置背景颜色提示，以便用户进行调整。这是一个很好的交互设计，在 [Ant Design](https://ant.design/docs/spec/feature-cn) 中也有提到这一点。
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/ant-design-input.png)

本文将介绍如何实现这一字数限制输入框，效果访问：[Codepen](https://codepen.io/yingshandeng/pen/boRNLx)
<!-- more -->

## 超出限制文字设置背景提示
实现这个输入框关键在于如何实现对超出字数限制部分的文字设置背景颜色，对于输入框一般有两种选择：
- `textarea` 多行文本输入
- `div` 设置 `contenteditable="true"`

`contenteditable="true"` 的 DIV 节点可以操作其中的 DOM 节点，那么我们可以监听输入区域，用户输入时，获取其中的 `innerText`，然后进行字数限制判断，设置其 `innerHTML`，划分两部分，超出部分设置背景颜色，代码大致如下：
```
<div contenteditable="true">
  文字，文字...
  <em>超出的文字...</em>
</div>

<script>
  var editorArea = document.querySelector('.editor-area')

  editorArea.addEventListener('input', (evt) => {
    var normalText;
    var exceedText;

    editorArea.innerHTML = normalText + '<em>' + exceedText + '</em>'
  })
</script
```
在每次输入后，都重新设置 `innerHTML`，会导致每次输入后，光标位置都跑到输入框起始位置了，看来这种方式不可行 😤

👉 解决方案：用两层 DIV 重叠
- 上层 DIV 节点用于文字输入，背景颜色透明
- 下层 DIV 节点用于高亮超出部分文字，文字颜色设置为透明，超出部分节点设置背景颜色

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/rich-input.png)

## 输入框 placeholder
`contenteditable="true"` 的 DIV 设置 placeholder 可以通过 CSS 来解决：
```
.editor-area[contenteditable=true]:empty:before {
  content: attr(placeholder);
  color: #ccd6dd;
  position: absolute;
}
```
通过选择器 `:empty` 判断输入框中内容是否为空，其实存在一些问题。点击回车的时候，会插入 `<div><br></div>` 或者 `<br>`，这部分会影响 `:empty` 的判断，譬如 DIV 中没有文字，但是存在一个换行时，这样 `:empty` 的判断就不会生效，placeholder 也就没有显示，所以我们可以通过另一种方式进行判断，通过 JS 手动添加、移除 class 类处理 placeholder 的显示和隐藏：

```
// CSS
.editor-area.is-showPlaceholder[contenteditable=true]:before {
  content: attr(placeholder);
  color: #ccd6dd;
  position: absolute;
}

// JS
editorArea.addEventListener('input', (evt) => {
  if (editorArea.innerHTML === '<div><br></div>' ||
    editorArea.innerHTML === '<br>' ||
    editorArea.innerHTML === '') {
    editorArea.classList.add('is-showPlaceholder')
  } else {
    editorArea.classList.remove('is-showPlaceholder')
  }
})
```

还有一种解决方式，第一层输入 DIV 节点使用 `textarea` 替换，`textarea` 中有 `placeholder` 属性

## 中文输入字数统计
在中文输入是，中文还未输入到输入框，字数就在统计了；合理的是，中文输入 composing 组合过程中不应进行字数统计，在中文输入到输入框后，才进行字数统计。
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/CCCDB51D-C8BD-4014-884B-C69A097F35C5.png)

解决这个问题需要用到 `compositionstart` 和 `compositionend` 这两个事件；前者表示输入组合开始，后者表示输入组合结束，在 `compositionend` 事件中就是我们字数统计的合理时机。
```
var isComposing = false

editorArea.addEventListener('compositionstart', () => {
  isComposing = true
})
editorArea.addEventListener('compositionend', () => {
  isComposing = false
  // 字数统计
  let text = editorArea.innerText
  setCounter(limitCnt - text.length)
})
```

🎏 如需考虑 emoji 字符长度计算的话，参考：[探究 emoji 字符长度](https://objcer.com/2017/07/20/explore-emoji-length/)

## 完整代码
效果访问：[Codepen](https://codepen.io/yingshandeng/pen/boRNLx)
```
<html>

<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
    html,
    body {
      display: flex;
      width: 100%;
      height: 100%;
      padding: 0;
      margin: 0;
      background: #E3F3FD;
      font-family: "Helvetica Neue",Helvetica,Arial,sans-serif;
    }

    .rich-editor {
      width: 508px;
      height: auto;
     /* max-height: 134px;
      overflow-y: scroll;*/
      margin: auto;
      padding: 8px;
      background: #fff;
      border-radius: 8px;
      border: 1px solid #A4D9F9;
      box-shadow: 0 0 0 1px #A4D9F9;
    }
    .wrapper {
      position: relative;
      width: 100%;
      min-height: 116px;
      z-index: 1;

    }
    .editor-area, .editor-backer {
      width: 100%;
      height: auto;
      min-height: 116px;

      font-size: 14px;
      line-height: 20px;
      word-wrap: break-word;
      background: transparent;
    }
    .editor-area {
      outline: none;
      border: none;

    }
    .editor-area.is-showPlaceholder[contenteditable=true]:before {
      content: attr(placeholder);
      color: #ccd6dd;
      position: absolute;
    }
    .editor-backer {
      position: absolute;
      left: 0;
      top: 0;
      color: transparent;
      z-index: -1;
    }
    .editor-backer em {
      background-color: #ffb8c2;
      font-style: normal;
      font-size: 14px;
      line-height: 20px;
      white-space: pre-wrap;
      word-wrap: break-word;
    }

    .counter {
      float: right;
      color: #657786;
      font-size: 14px;
      line-height: 20px;
      text-align: right;
      user-select: none;
    }
    .counter.max-reached {
      color: #e0245e;
    }
    </style>
</head>

<body>
    <div class="rich-editor">
      <div class="wrapper">
        <div class="editor-area" contenteditable="true" placeholder="Enter text here..."></div>
        <div class="editor-backer"></div>
      </div>
      <span class="counter">140</span>
    </div>

    <script>
      var editorArea = document.querySelector('.editor-area')
      var editorBacker = document.querySelector('.editor-backer')
      var counter = document.querySelector('.counter')
      var limitCnt = 140
      var isComposing = false

      editorArea.addEventListener('compositionstart', () => {
        isComposing = true
      })
      editorArea.addEventListener('compositionend', () => {
        isComposing = false
        //
        let text = editorArea.innerText
        setCounter(limitCnt - text.length)
      })

      function setCounter (cnt) {
        counter.innerHTML = cnt
        if (cnt < 0) {
          counter.classList.add('max-reached')
        } else {
          counter.classList.remove('max-reached')
        }
      }

      var inputCompose = function () {
        if (editorArea.innerHTML === '<div><br></div>' ||
          editorArea.innerHTML === '<br>' ||
          editorArea.innerHTML === '') {
          editorArea.classList.add('is-showPlaceholder')
        } else {
          editorArea.classList.remove('is-showPlaceholder')
        }

        let text = editorArea.innerText
        let remainingCnt = limitCnt - text.trim().length
        if (remainingCnt < 0) {
          let allowedText = text.substring(0, limitCnt)
          let refusedText = text.substring(limitCnt)

          editorBacker.innerHTML = allowedText + '<em>' + refusedText + '</em>'
        } else {
          editorBacker.innerHTML = ''
        }

        if (!isComposing) {
          setCounter(remainingCnt)
        }
      }

      inputCompose();
      editorArea.oninput = inputCompose
    </script>
</body>

</html>
```
