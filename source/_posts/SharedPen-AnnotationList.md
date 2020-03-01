title: SharedPen 之 AnnotationList
date: 2018-02-27 22:35:29
tags: AnnotationList
categories: SharedPen
---

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/SharedPen-Annotation.png)

CodeMirror 是一款优秀的开源文本编辑器，常用于代码编辑器。但并不支持富文本编辑功能。但是得益于 CodeMirror 的 API
> [markText](https://codemirror.net/doc/manual.html#api_marker): **Can be used to mark a range of text with a specific CSS class name.**

能够为指定 range 的文本设置一个 CSS class，这样我们就可以通过这个 class 来设置富文本样式。例如对于文本：`ABCDEFGHF`，我们可以通过如下方式为其设置 class
```
doc.markText({line:0, ch: 0}, {line: 0, ch: 3}, {className: 'classA'})
doc.markText({line:0, ch: 3}, {line: 0, ch: 6}, {className: 'classB'})
doc.markText({line:0, ch: 6}, {line: 0, ch: 9}, {className: 'classC'})
```

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/marktext.png)

`markText` 方法会返回一个 `CodeMirror.TextMarker` 对象，该对象中的 `clear()` 方法用于清除 class，这样就非常便于我们动态为文本设置富文本样式了。


为了管理，操作整个富文本文档流，我们引入 **AnnotationList** 单链表结构，把每一段富文本当成 `Node` 节点，由此形成链表结构。
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/annotation-list.png)

本文将详解 AnnotationList 单链表结构 🤓

<!-- more -->
## 文本区间 `Span` 类
```
class Span {
  constructor (pos, length) {
    this.pos = pos
    this.length = length
  }
  end () {
    return this.pos + this.length
  }
}
```

`Span` 表示一段文本区间：
- `pos`：区间起点
- `length`：区间长度（若长度为0，即未选中文字，只表示光标位置）

## 链表节点 `Node` 类
```
class Node {
  constructor (length, annotation) {
    this.length = length
    this.annotation = annotation
    this.attachedObject = null
    this.next = null
  }
}
```

链表节点包含四个属性：
- `length`：表示该节点的文本的长度，需要注意的是，**节点并不需要直接存储文本内容**
- `annotation`：表示节点包含的富文本属性
- `attachedObject`：`CodeMirror.TextMarker` 对象
- `next`：指向下一个节点

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/list-node.png)

## 辅助方法 `getAffectedNodes_(span)`
此方法用于获取指定文本区间(span)在单链表中的节点信息，包括：
- `start`: the node contains the first character in span.
- `end`: the node contains the last character in span.
- `beforeStart`: the node before `start` node.
- `succ`: the node after `end` node if `span.end()` was on a node boundary, else `null`.
- `pred`: the node before `start` if `span.pos` was on a node boundary, else `null`.
- `beforePred`: the node before `pred` node.
- `startPos`: the position of `start` node
- `predPos`: the position of `pred` node

### `start` 和 `end`
`start` 和 `end` 分别表示文本区间(span)的第一个和最后一个字符所在的节点。
大致有如下三种情况：

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/start-end-node.png)

### `pred` 和 `succ`
`pred` 和 `succ` 是指文本区间(span)左右边界恰好也是处于单链表节点边界处时，左边界的前一个节点和右边界的后一个节点。
大致有如下五种情况：

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/pred-succ-node.png?t=1)

### `startPos` 和 `predPos`
`startPos` 和 `predPos` 是指 `start` 和 `pred` 节点首字符在整个文档中的位置

## 链表操作
文档编辑过程中的富文本操作对应到单链表中的操作包括增删改查，其中插入，删除和更新，这些方法都依赖这个方法：
```js
wrapOperation_ (span, operationFn)

this.wrapOperation_(span, function (startPos, start) {
    return newNodes
  }
})
```

其中 `span` 指定文本区间；而 `operationFn(res.startPos, res.start)` 回调方法根据从辅助方法 `getAffectedNodes_(span)` 获取的 **`startPos`, `start`** 信息生成一段新的链表段。`wrapOperation_` 将新生成的这段链表融入到原有的 AnnotationList 单链表。

### 插入操作 `insertAnnotatedSpan`
插入操作包括两种情况：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/insert-span.png)

其中前者在节点间隔处插入，`start` 节点为 `null`，此时由新插入的文本生成 Node 节点作为新生成链表段直接返回；而后者在节点中插入，`start` 节点为当前节点，生成新链表段需要将 `start` 节点从插入点分裂生成两个节点，新插入的文本作为新 Node 节点插入其中，如下图所示：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/insert-span-action.png?t=1)

```js
function insertAnnotatedSpan (span, annotation) {
  this.wrapOperation_(new Span(span.pos, 0), function (oldPos, old) {
    var toInsert = new Node(span.length, annotation)
    if (!old) {
      return toInsert
    } else {
      var newNodes = new Node(0, NullAnnotation)
      // Insert part of old before insertion point.
      newNodes.next = new Node(span.pos - oldPos, old.annotation)
      // Insert new node.
      newNodes.next.next = toInsert
      // Insert part of old after insertion point.
      toInsert.next = new Node(oldPos + old.length - span.pos, old.annotation)
      return newNodes.next
    }
  })
}
```

### 删除操作 `removeSpan`
删除操作中，要求文本区间长度大于 0；其执行过程如下图所示：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/delete-span-action.png)

```js
function removeSpan (removeSpan) {
  if (removeSpan.length === 0) { return }
  this.wrapOperation_(removeSpan, function (oldPos, old) {
    var newNodes = new Node(0, NullAnnotation)
    var current = newNodes
    // ① Add new node for part before the removed span (if any).
    if (removeSpan.pos > oldPos) {
      current.next = new Node(removeSpan.pos - oldPos, old.annotation)
      current = current.next
    }

    // ② Skip over removed nodes.
    while (removeSpan.end() > oldPos + old.length) {
      oldPos += old.length
      old = old.next
    }

    // ③ Add new node for part after the removed span (if any).
    var afterChars = oldPos + old.length - removeSpan.end()
    if (afterChars > 0) {
      current.next = new Node(afterChars, old.annotation)
    }
    return newNodes.next
  })
}
```

### 更新操作 `updateSpan`
更新操作中，文本区间长度也要大于 0；其执行过程如下图所示：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/update-span-action.png)

```js
function updateSpan (span, updateFn) {
  if (span.length === 0) { return }
  this.wrapOperation_(span, function (oldPos, old) {
    var newNodes = new Node(0, NullAnnotation)
    var current = newNodes
    var currentPos = oldPos

    // ① Add node for any characters before the span we're updating.
    var beforeChars = span.pos - currentPos
    if (beforeChars > 0) {
      current.next = new Node(beforeChars, old.annotation)
      current = current.next
      currentPos += current.length
    }

    // ② Add updated nodes for entirely updated nodes.
    while (old !== null && span.end() >= oldPos + old.length) {
      var length = oldPos + old.length - currentPos
      current.next = new Node(length, updateFn(old.annotation, length))
      current = current.next
      oldPos += old.length
      old = old.next
      currentPos = oldPos
    }

    // ③ Add updated nodes for last node.
    var updateChars = span.end() - currentPos
    if (updateChars > 0) {
      current.next = new Node(updateChars, updateFn(old.annotation, updateChars))
      current = current.next
      currentPos += current.length

      // ④ Add non-updated remaining part of node.
      current.next = new Node(oldPos + old.length - currentPos, old.annotation)
    }
    return newNodes.next
  })
}
```

### wrapOperation_
`wrapOperation_` 方法是 AnnotationList 链表操作的核心方法，通过这个方法会从单链表中提取需旧节点数组和新节点数组，如下图所示：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/oldNodes-newNodes.png)

其中浅灰色层覆盖的节点为 `oldNodes` 数组；浅蓝色层覆盖的节点为 `newNodes` 数组。得到这两个数组后，首先对 `oldNodes` 数组中的节点进行操作，通过 `attachedObject` 属性得到 `CodeMirror.TextMarker` 对象，调用 `clear()` 方法，取消先前设置的 class。然后对 `newNodes` 数组中的节点，通过 `markText` 方法设置相应的 class。

👇下面列几个在这个方法中比较关键(链表优化)的处理：

**1. `mergeNodesWithSameAnnotations_`**
通过 `wrapOperation_ (span, operationFn)` 回调函数 `operationFn` 会生成一小段新的链表 `var newSegment = operationFn(res.startPos, res.start)`，得到这段新链表段后，需要对其进行一个预处理，合并其中节点富文本属性 **`annotation`** 相同的节点，这个过程调用 `mergeNodesWithSameAnnotations_` 方法
```
this.mergeNodesWithSameAnnotations_(newSegment)
```

**2. 存在 `pred` 或者 `succ` 节点时的判断**
考虑如下这种情况：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/AD4C499B-D669-4251-B59D-0162041C4D97.png)

对中间三个节点进行更新操作，那么由于选择的文本区间边界正好处于链表的节点边界，所以 `pred` 和 `succ` 节点都不为空，那么此时需要进行两个判断：
- `pred` 节点和 `newSegment` 的首节点判断富文本属性 `annotation` 是否相同；若相同，说明 `pred` 节点和`newSegment` 的首节点可以**合并**，那么需要将 `pred` 节点纳入 `oldNodes` 数组
- 同理，`succ` 节点和 `newSegment` 的最后一个节点也需要执行如上逻辑的判断

```
if (res.pred && res.pred.annotation.equals(newSegment.annotation)) {
  // We can merge the pred node with newSegment's first node.
  includePredInOldNodes = true
}

if (res.succ && res.succ.annotation.equals(newSegment.annotation)) {
  // We can merge newSegment's last node with the succ node.
  includeSuccInOldNodes = true
}
```

### 查询操作
**AnnotationList** 类中也提供了两个链表查询方法，根据 `pos` 位置信息或者 `span` 指定文本区间查询链表节点信息，基本上都是遍历链表的操作，此处不详述，直接参考源码即可。
```
function getAnnotatedSpansForPos (pos)
function getAnnotatedSpansForSpan (span)
```

## 思考
**AnnotationList** 单链表结构执行操作的效率比较低，所有操作的算法复杂度都是 `O(n)`，这个链表长度会随着文章长度，富文本属性设置操作的增加而变长。优化的方向是，**将链表结果改造成树状结构**。这部分优化内容在文章 [SharedPen 之区间树优化](https://objcer.com/2018/03/05/SharedPen-IntervalTree-Optimization/) 进行了详细介绍，欢迎前往阅读。🎏

