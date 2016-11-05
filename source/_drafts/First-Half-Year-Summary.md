title: First Half Year Summary
date: 2016-08-08 15:02:51
tags:
categories:
---



> Project Summary

## CKEditor

#### 关于撤销和回撤 undo/redo

CKEditor 自带 undo plugin, 但是直接添加到项目中后，发现并不能正常的工作。

1、undo 工作原理

<!-- ![undo](/Users/YingshanDeng/Desktop/undo.png) -->

维持一个 snapshots 数组（栈），数组中每一个元素都是编辑区某一状态时的全量数据。

其中 Image 这个数据结构代表某一时刻下编辑区中的数据。

。。。

调用 `save` 函数即可以向 snapshots 数组中添加元素。其中大致的逻辑如下：



我们编辑文本后，并不会马上调用 save 函数，而是在进行undo, redo,即调用 undo 和 redo 时，才会进行 save 操作，然后再执行 undo 和 redo 有关的逻辑



关于键盘输入，区分两种：①删除键(Functional) ②可打印(Printable)

strokesRecorded （[0, 0]）数组分别保存二者输入的次数，超过 strokesLimit 置为0

### CKEditor plugin







CKEditor builder node

https://www.npmjs.com/package/node-ckbuilder



CKEditor 5 ： https://medium.com/content-uneditable/ckeditor-5-the-future-of-rich-text-editing-2b9300f9df2c#.6ct0zzvnw