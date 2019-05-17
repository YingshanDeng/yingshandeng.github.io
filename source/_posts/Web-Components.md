title: Web Components
date: 2017-10-20 16:35:02
tags: Web Components
categories: Web Components
---


![](http://cdn.objcer.com/web-components.png)

组件是构建现代网页应用的基础。组件化给前端开发带来了极大的效率提升，提高了代码的重用性；组件化的 UI 框架也层出不穷，React, Vue 等等。但是这些框架缺乏标准，代码层面可能存在很大的差异，如何对前端组件标准化就显得尤为重要。
Web Components 作为面向未来的 Web 组件化标准，包括 HTML Templates, Shadow DOM, Custom elements 和 HTML Imports 四部分，浏览器原生支持，所以 Web Components 是最彻底的组件化解决方案。

<!-- more -->

## Web Components
Web Components 包括 HTML Templates, Shadow DOM, Custom elements 和 HTML Imports 四部分， 需要浏览器支持，但是目前为止，未被所有浏览器完整实现。
参考 [https://www.webcomponents.org/](https://www.webcomponents.org/) 给出的浏览器支持情况：

![](http://cdn.objcer.com/web-component-browser-support.png)

基于 Web Components，Google 推出了 [Polymer](https://www.polymer-project.org/) 框架，作为应用层框架，使用 Polymer 相较于直接使用 Web Components 接口，开发网页应用更加方便简洁，提高效率。Polymer 在今年 5 月份，推出 [Polymer 2.0](https://www.polymer-project.org/blog/2017-05-15-time-for-two.html)，使用 Shadow DOM v1 和 Custom elements v1，对于不兼容的浏览器，也提供了 polyfill。而在今年 Polymer Summit 大会上，推出了最新 [Polymer 3.0 preview](https://www.polymer-project.org/blog/2017-08-22-npm-modules.html)，也指明了 Polymer 最新的发展方向，其中主要变化是：① 从 bower 转向 npm ② 使用 ES6 modules，弃用 HTML Imports

[Building Components](https://developers.google.com/web/fundamentals/web-components/) 系列文章中详细介绍了 Custom Elements 和 Shadow DOM，阅读后整理如下：
- [Custom Elements](https://objcer.com/2017/10/20/Custom-Elements/)
- [Shadow DOM](https://objcer.com/2017/10/20/Shadow-DOM/)

通过 Web Components 开发自定义组件就是用 Custom Elements 做外壳，其中建立 Shadow DOM 提供 DOM 封装，作用域 CSS 等特性，通过 template 模板构建 DOM，每一个组件都是一个 html 文件，使用时通过 HTML Import引入。

关于 Web Components 网上也有很多讨论质疑的声音：
[精读《Web Components 的困境》](https://github.com/dt-fe/weekly/issues/15)

## Polymer
Polymer 是基于 Web Components 的组件化框架，了解 Web Components 对于理解使用 Polymer 有很大的帮助。Polymer 可以划分为以下四大部分内容：
- Custom elements
- Shadow DOM
- Events
- Data system

关于 Polymer 的学习会通过下一个系列文章进行记录。

