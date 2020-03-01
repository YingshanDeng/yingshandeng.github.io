title: Javascript DOM 事件模型
date: 2017-08-14 19:39:01
tags: [事件捕获, 事件冒泡]
categories: JS
---

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/js-event.jpg)
<!-- more -->

## Bubbling and capturing
Javascript DOM 事件流存在如下三个阶段：
- 事件捕获阶段 Capturing phase – **the event goes down to the element.**
- 处于目标阶段 Target phase – the event reached the target element.
- 事件冒泡阶段 Bubbling phase – **the event bubbles up from the element.**

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/eventflow@2x.png)

**Javascript DOM 标准事件流的触发的先后顺序为：先捕获再冒泡。**点击 `<td>` DOM 节点时，事件传播顺序：首先是事件捕获阶段，从上向下传播；然后到达点击事件目标节点；最后是冒泡阶段，从下向上传播。

```
addEventListener(type: DOMString, callback: EventListener, capture?: boolean)
```
DOM 节点添加事件监听方法 `addEventListener` 中第三个参数可以指定是该监听是添加在事件捕获阶段或者是事件冒泡阶段，默认为 `false`，即事件冒泡阶段；显式指定为 `true`，即事件捕获阶段。

实际应用中，为 DOM 元素添加事件监听绝大多数都是添加到事件冒泡阶段，很少会用到事件捕获。我们通过 `on<event>-property` HTML 属性添加的事件监听默认也是添加到事件冒泡阶段

注意到事件流处理有三个阶段，其中第二个阶段：处于目标阶段，并不单独处理，事件捕获阶段和冒泡阶段的添加的监听处理就包含了这个阶段。

### 并非所有的事件都支持冒泡
> ❗ **Almost** all events bubble.

注意关键字 “almost”，所以并不是所有的事件都支持冒泡。在 wiki [DOM_events](https://en.wikipedia.org/wiki/DOM_events#Events) 中我们就可以找到一些不支持冒泡的事件，例如 `focus`、`blur` 等等
```
a.addEventListener('click', (event) => {
    console.log(event.bubbles) // > true
})

a.addEventListener('focus', (event) => {
    console.log(event.bubbles) // > false
})
```
我们可以通过 `event.bubbles` 来判断是否支持事件冒泡

### 三个常用方法

- `event.stopPropagation` 阻止捕获和冒泡阶段中当前事件的进一步传播
在事件监听回调中调用此方法，若是捕获阶段，则停止向下传递事件；若是冒泡阶段，则停止向上传递事件。

- `event.stopImmediatePropagation` 阻止调用相同事件的其他侦听器
如果某个元素有多个相同类型事件的事件监听函数, 则当该类型的事件触发时, 多个事件监听函数将按照顺序依次执行. 如果某个监听函数执行了 `event.stopImmediatePropagation()` 方法, 则除了该事件的冒泡行为被阻止之外(event.stopPropagation方法的作用), 该元素绑定的后序相同类型事件的监听函数的执行也将被阻止。例子：
```
<script>
    document.querySelector("p").addEventListener("click", function(event)
    {
        alert("我是p元素上被绑定的第一个监听函数");
    }, false);
    document.querySelector("p").addEventListener("click", function(event)
    {
        alert("我是p元素上被绑定的第二个监听函数");
        event.stopImmediatePropagation();
        //执行stopImmediatePropagation方法,阻止click事件冒泡,并且阻止p元素上绑定的其他click事件的事件监听函数的执行.
    }, false);
    document.querySelector("p").addEventListener("click", function(event)
    {
        alert("我是p元素上被绑定的第三个监听函数");
        //该监听函数排在上个函数后面,该函数不会被执行.
    }, false);
    document.querySelector("div").addEventListener("click", function(event)
    {
        alert("我是div元素,我是p元素的上层元素");
        //p元素的click事件没有向上冒泡,该函数不会被执行.
    }, false);
</script>
```

- `event.preventDefault` 如果事件可取消，则取消该事件，而不停止事件的进一步传播。
    - 在事件触发后的任何阶段调用 `preventDefault` 方法来取消该事件, **意味着该事件的所有默认动作都不会发生.** 例如可以利用 `preventDefault()` 方法来阻止一个 `input` 元素内非法字符的输入等等
    - 调用事件的 `preventDefault()` 方法后,会引起该事件的 `event.defaultPrevented` 属性值变为 `true`
    - 可以查看 `event.cancelable` 属性来判断一个事件的默认动作是否可以被取消. 在 `cancelable` 属性为 `false` 的事件上调用 `preventDefault` 方法没有任何效果
    - `preventDefault` 方法**不会阻止该事件的进一步冒泡**. `event.stopPropagation` 方法才有这样的功能

### `event.target` 和 `event.currentTarget`
> The most deeply nested element that caused the event is called a target element, accessible as event.target.
Note the differences from this (=event.currentTarget):
- event.target – is the “target” element that initiated the event, it doesn’t change through the bubbling process.
- this – is the “current” element, the one that has a currently running handler on it.

- `event.target` – 指向触发事件的元素，在事件冒泡过程中该值不变
- `event.currentTarget` = this – 事件绑定的当前元素

只有被点击的那个目标元素的 `event.target` 才会等于 `event.currentTarget`，看如下例子
```
<div class="outer">
  outer
  <div class="inner">inner</div>
</div>

<script>
  var outerEl = document.querySelector('.outer');
  var innerEl = document.querySelector('.inner');

  innerEl.addEventListener('click', (event) => {
    console.log('inner', event.target, event.currentTarget)
  });
  outerEl.addEventListener('click', (event) => {
    console.log('outer', event.target, event.currentTarget)
  });
</script>
```
点击 inner 节点，执行结果：
```
inner <div class=​"inner">​inner​</div>​ <div class=​"inner">​inner​</div>​
outer <div class=​"inner">​inner​</div>​ <div class=​"outer">​…​</div>​
```

## 执行顺序的问题
我们知道，Javascript DOM 标准事件流的触发的先后顺序为：先捕获再冒泡。如果 DOM 节点同时绑定两个事件监听，一个用于捕获阶段，一个用于冒泡阶段，两个事件的执行顺序真的如此么？
```
<div class="outer">
  element1
  <div class="inner">element2</div>
</div>
```

```
-----------------------------------
| element1                        |
|   -------------------------     |
|   |element2               |     |
|   -------------------------     |
|                                 |
-----------------------------------
```
分别为内外两个元素添加两个点击事件，一个用于捕获阶段，一个用于冒泡阶段
```
<script>
  var outerEl = document.querySelector('.outer');
  var innerEl = document.querySelector('.inner');

  innerEl.addEventListener('click', (event) => {
    console.log('child bubble')
  }, false);
  innerEl.addEventListener('click', (event) => {
    console.log('child capture')
  }, true);

  outerEl.addEventListener('click', (event) => {
    console.log('parent bubble')
  }, false);
  outerEl.addEventListener('click', (event) => {
    console.log('parent capture')
  }, true);
</script>
```

根据前面的知识，当点击 element2 元素和 element1 元素的时候，我们应该会得到：
```
#点击 element2 元素
parent capture
child capture
child bubble
parent bubble

#点击 element1 元素
parent capture
parent bubble
```
**❗️但是，**实际上我们点击 element2 和点击 element1 的时候，得到的结果却是（其中都出现了先 bubble 后 capture）：
```
# 点击 element2
parent capture
child bubble
child capture
parent bubble

# 点击 element1
parent bubble
parent capture
```

这是为什么呢？这跟前面的结论不符啊 🤔 通过观察我们可以发现：
① DOM 元素先添加了用于冒泡阶段的事件监听，后添加了用于捕获阶段的事件监听
② 被点击元素先执行了冒泡阶段的事件监听，后执行捕获阶段的事件监听；而点击事件的父节点事件监听执行顺序正常
③ 我们猜测是否跟事件监听添加顺序有关系，我们调换一下添加顺序：
```
<script>
  var outerEl = document.querySelector('.outer');
  var innerEl = document.querySelector('.inner');

  innerEl.addEventListener('click', (event) => {
    console.log('child bubble')
  }, true);
  innerEl.addEventListener('click', (event) => {
    console.log('child capture')
  }, false);

  outerEl.addEventListener('click', (event) => {
    console.log('parent bubble')
  }, true);
  outerEl.addEventListener('click', (event) => {
    console.log('parent capture')
  }, false);
</script>
```
发现执行得到的结果正常

**❗️给出结论：**
- 绑定在被点击元素的事件是按照代码添加顺序执行，其他元素先捕获后冒泡
- **所以事件的执行顺序是：父元素的捕获阶段事件 -> 触发事件元素按代码添加顺序的事件 -> 父元素的冒泡阶段事件**

## 参考链接
[Bubbling and capturing](https://javascript.info/bubbling-and-capturing)
