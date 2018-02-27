title: SharedPen 富文本实时协同编辑器
date: 2018-02-27 22:39:33
tags: ['富文本', '实时协同编辑器']
categories: SharedPen
---

![](http://7vikhl.com1.z0.glb.clouddn.com/SharedPen-Main.png)

2017 年末，公司组织了 WPS 黑客马拉松比赛已正式开赛，比赛的主题是：利用 WPS 客户端、云，及其它第三方开放服务，针对文档的查看 、创作 ，以及工作、学习中的团队分享、合作等，来构想并实现 WPS 产品创新点。冲着主题宽泛，开发时间长，且一等奖 1 万的奖金 🤑，毫不犹豫的报名了。经过一个多月的开发，SharedPen 富文本实时协同编辑器基本功能开发完成，最终在评审中也如愿获得一等奖 👏
SharedPen 目前仍处于开发阶段，欢迎感兴趣的朋友与我联系一起交流，项目地址：[GitHub: YingshanDeng/SharedPen](https://github.com/YingshanDeng/SharedPen)。本文将对 SharedPen 整体架构进行介绍，后续会有系列文章介绍其中具体的技术实现细节(敬请期待 🎏)。

<!-- more -->

## 关于协同编辑器
协同编辑也就是多个人对一份文档进行编辑，每个协作这都有自己的输入光标，并可以可以自由地和其他协作者一起输入。对于 web-based 的协同编辑器，目前有很多成熟的产品。像我很喜欢用的: [Google Docs](https://www.google.com/docs/about/)，国内的: [石墨文档](https://shimo.im/)。

另一个值得介绍的是 [Teletype](https://teletype.atom.io/)，这是在旧金山举办的 2017 QCon 大会上，GitHub 发布的 Atom 实时协作插件，Teletype 可以让两名开发者轻松的协作编写代码，由于 Atom 是基于 [Electron](https://electronjs.org/) 构建的应用程序，所以我们也可以将其看作是 web-based 协同编辑器。

**协同编辑的关键是要解决数据一致性的问题**，不同的协作者修改他们自己的文档副本，本地的编辑会立刻应用到本地副本，之后会传输给其他协作者，这会导致不同的副本可能会以不同的顺序应用各种修改，应该有算法来确保在协同编辑之后，所有副本最终的内容是一致的。

## 数据一致性解决方案
首先介绍一下分布式系统，分布式系统即运行在多物理计算机上的系统。这个是人类目前实际可行的构建超大规模系统的唯二办法之一（另一种就是构建超级计算机）。构建一个分布式系统的难点在于：让分布式系统运作起来正确性与单机程序完全无二。而协同编辑器，我们也可以看做是一个分布式系统，多端的协作者修改文档，而最终文档内容要收敛到相同的状态
![](http://7vikhl.com1.z0.glb.clouddn.com/real-time-portals-fa6fa303e261b1679024081d6229c9f9.png)

目前来说，解决分布式系统数据一致性问题，主流的有两大方案：
- **Conflict-free replicated data type** (CRDT)
- **Operational transformation** (OT)

### Conflict-free replicated data type
Conflict-free replicated data type，简称为 CRDT，直译的话即是：无冲突可复制数据类型。
> Wikipedia 介绍: [Conflict-free replicated data type](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type)
In distributed computing, a conflict-free replicated data type (CRDT) is a **data structure** which can be replicated across multiple computers in a network, where the replicas can be updated independently and concurrently without coordination between the replicas, and where it is always mathematically possible to resolve inconsistencies which might result.

简单的理解：CRDT 就是这样一些适应于分布式系统的可以保持最终一致性的**数据结构**的统称。GitHub 开源的 Teletype 协同编辑插件，使用的就是 CRDT 算法。GitHub 为此开发的 CRDT 是一个独立的库，有兴趣可前往 GitHub 具体了解。
> 项目地址：[atom/teletype-crdt](https://github.com/atom/teletype-crdt) String-wise sequence CRDT powering peer-to-peer collaborative editing in Teletype for Atom.

### Operational transformation
Operational transformation，简称 OT，直译的话即是：操作转换
OT 它起源于 1989 年发表的一篇[研究论文](https://dl.acm.org/citation.cfm?doid=67544.66963)，进过多年的发展，现在已经是非常成熟的一致性解决算法，很多协同编辑器都采用此算法作为协同解决方案，例如我们熟知的 Google Docs。
> Wikipedia 介绍: [Operational transformation](https://en.wikipedia.org/wiki/Operational_transformation)
Operational transformation (OT) is a technology for supporting a range of collaboration functionalities in advanced collaborative software systems. OT was originally invented for consistency maintenance and concurrency control in collaborative editing of plain text documents. Two decades of research has extended its capabilities and expanded its applications to include group undo, locking, conflict resolution, operation notification and compression, group-awareness, HTML/XML and tree-structured document editing, collaborative office productivity tools, application-sharing, and collaborative computer-aided media design tools (see OTFAQ). In 2009 OT was adopted as a core technique behind the collaboration features in Apache Wave and Google Docs.

**SharedPen 也是采用 OT 作为协同编辑解决方案**，使用了 [ot.js](https://github.com/Operational-Transformation/ot.js) 库，OT 算法将有另一篇文章单独介绍，敬请期待 😋

## 富文本编辑器
SharedPen 是基于 web 端的所见即所得 (WYSIWYG) 编辑器。
![](http://7vikhl.com1.z0.glb.clouddn.com/sharedpen.png)

在 ot.js 库中，使用 [CodeMirror](https://codemirror.net/) 作为文本编辑器，CodeMirror 是一款优秀的编辑器，多用于代码编辑器，譬如在 Chrome 浏览器的调试器中，就使用 CodeMirror 作为代码编辑器。ot.js 库基于 CodeMirror 实现了一个简单的纯文本编辑器，demo 可以参考：[GitHub: ot.js-demo](https://github.com/YingshanDeng/ot.js-demo)

CodeMirror 编辑器具有丰富友好的 API 接口：
- **坐标系统**
  CodeMirror 中以行为单位，通过 `{line, ch}` 第几行的第几个字符可以准确为每一个字符定位 `pos`，同时每一个字符都有一个全局索引 `index`，通过 `doc.posFromIndex` 和 `doc.indexFromPos` 两个辅助方法可以对二者进行转换
- **丰富的事件回调: `change`, `beforeChange`, `cursorActivity`...**
其中编辑事件 `change` 的回调参数信息非常丰富，便于我们将其转换成 OT 算法中的 operation
![](http://7vikhl.com1.z0.glb.clouddn.com/codemirror-change-insert.png)
![](http://7vikhl.com1.z0.glb.clouddn.com/codemirror-change-delete.png)
- **`markText` 富文本支持 API**
  注意到 CodeMirror 其实并不支持富文本编辑，但是得益于 API `markText` 可以动态为指定范围的文本设置一个 CSS class，也可以通过自身实现为其增加富文本编辑功能，这部分内容在文章：👉 [SharedPen 之 AnnotationList](https://objcer.com/2018/02/27/SharedPen-AnnotationList/) 中将进行详细介绍

## 架构
 | 架构 | Server 作用
--- | --- | ---
Google Docs，石墨文档 | C/S 模式 | 负责转发多端操作和数据持久化存储
Teletype(Atom协同编辑插件) | Peer-to-Peer 模式 | 辅助建立连接(握手)

### Peer-to-Peer 模式
Atom 协同编辑插件 Teletype，采用 Peer-to-Peer 模式，通过 WebRTC 加密多端之间的通信。除了最初的握手依赖于 GitHub 的服务器之外，所有的传输都是点对点的（不需要访问 GitHub 服务器）。
![](http://7vikhl.com1.z0.glb.clouddn.com/sharedpen-peer-to-peer.png?t=1)

### C/S 模式
像 Google Docs，石墨文档都是采用 C/S 模式作为前后端架构，SharedPen 也是采用这种架构模式，前后端建立 socket 连接，相互实时发送 operation 操作，服务端中还对数据进行持久化存储。
![](http://7vikhl.com1.z0.glb.clouddn.com/sharedpen-client-server.png)

## 参考链接
[谈谈CRDT](http://liyu1981.github.io/what-is-CRDT/)
[分布式CRDT模型](http://www.jdon.com/artichect/crdt.html)
[What’s different about the new Google Docs?](https://drive.googleblog.com/2010/05/whats-different-about-new-google-docs.html)
