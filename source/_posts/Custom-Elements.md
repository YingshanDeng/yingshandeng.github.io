title: Custom Elements
tags: Custom Elements
date: 2017-10-20 16:07:56
categories: Web Components
---

![](http://7vikhl.com1.z0.glb.clouddn.com/custom-elements.png)

Custom Elements 为开发者提供了创建新 HTML 标记，扩展现有 HTML 标记的能力，创建自定义新的元素，将 JS 逻辑行为和元素 DOM 关联起来，代码模块化，提供重用性。

<!-- more -->

## 自定义元素
**创建自定义元素包括两个步骤：**
- 通过 class **定义**自定义元素
  ```
  class AppDrawer extends HTMLElement {
    constructor() {
      // If you define a ctor, always call super() first!
      super();
      ...
    }
  }
  ```
- 向浏览器**注册**该自定义元素
  ```
  class AppDrawer extends HTMLElement {...}
  window.customElements.define('app-drawer', AppDrawer);

  // 匿名类
  window.customElements.define('app-drawer', class extends HTMLElement {...});
  ```

**创建自定义元素几点注意：**
- 自定义元素定义类中的构造函数中，总是需要调用 `super()`
- 自定义元素名称必须要包含一个短横线(-)，浏览器以次来区分自定义元素和常规元素。例如 `<x-tags>`、`<my-element>` 为有效名称；而 `<tabs>` 和 `<foo_bar>` 则为无效名称
- 不能多次注册同一自定义元素，否则会抛出 `DOMException` 异常
  ```
  class A extends HTMLElement{}
  window.customElements.define('app-drawer', A);

  class B extends HTMLElement{}
  window.customElements.define('app-drawer', B); // ❗️DOMException
  ```
- 自定义元素不能自闭合(self-closing)，所以需要写结束标签，例如 `<x-tags></x-tags>`

**定义新的自定义元素，class 类继承 `HTMLElement`；也可以扩展已有的元素，包括：**
- 扩展自定义元素
  ```
  class FancyDrawer extends AppDrawer {
    constructor() {
      super();
      ...
    }
  }
  ```
- 扩展 HTML 原生元素
  ```
  class FancyButton extends HTMLButtonElement {
    constructor() {
      super();
      ...
    }
  }

  customElements.define('fancy-button', FancyButton, {extends: 'button'});
  ```
  扩展 HTML 原生元素时，对 `define()` 的调用会稍有不同，需要传入第三个参数告知浏览器要扩展的标记。

使用自定义元素与 `<div>` 或任何其他元素的使用没有区别。可以在页面上声明，也可也通过 JavaScript 动态创建实例，也可添加事件侦听器，诸如此类。
```
<app-drawer></app-drawer>
let drawer = new AppDrawer()

// ❗️自定义 HTML 原生元素需要在原生标记上添加 is="" 属性来声明
<button is="fancy-button">Fancy button!</button>
let button = new FancyButton()

// customElements.get 方法通过传入自定义元素标记名称，获取元素的构造函数
let Drawer = customElements.get('app-drawer');
let drawer = new Drawer();
```

## 自定义元素的响应
自定义元素提供了一些响应方法，表征其生命周期的不同时刻，我们可以在这些响应方法（回调方法）中执行相关的逻辑。

 名称 | 调用时机
---- | ----
constructor | 创建或升级元素的一个实例。用于初始化状态、设置事件侦听器或创建 Shadow DOM
connectedCallback | 元素每次插入到 DOM 时都会调用
disconnectedCallback | 元素每次从 DOM 中移除时都会调用
attributeChangedCallback(attrName, oldVal, newVal) | 属性添加、移除、更新或替换时调用；创建元素或者升级时，也会调用它来获取初始值
adoptedCallback | 自定义元素被移入新的 document （例如，有人调用了 document.adoptNode(el)）

```
class AppDrawer extends HTMLElement {
  constructor() {
    super(); // always call super() first in the ctor.
    ...
  }
  connectedCallback() {
    ...
  }
  disconnectedCallback() {
    ...
  }
  attributeChangedCallback(attrName, oldVal, newVal) {
    ...
  }
}
```

- `attributeChangedCallback` 响应仅针对 `observedAttributes` 属性中列出的，对于 class, style 等属性的修改并不会调用该响应，这也是处于性能考虑
- `disconnectedCallback()` 响应主要用于清理工作，例如移除事件侦听器等。但是如果用户关闭了标签，`disconnectedCallback()` 响应将无法调用
- **响应是同步的**，例如移除自定义元素后，会立即收到 `disconnectedCallback()` 响应

## 自定义元素升级(upgrades)
**自定义元素可以在其定义注册之前使用。**

HTML 解析标签容错能力强，对于页面上声明的 `<asdfasdf>` 这类未知的标签，浏览器也能接受，因为 HTML 规范允许这类未知标签，它把这类未知标签生成的元素当成 `HTMLUnknownElement` 进行解析。而当页面中存在符合自定义元素命名规则（包含短横线'-'）的标签，但是还未定义注册时，浏览器将其视为潜在的自定义元素，解析成 `HTMLElement`

```
// "tabs" is not a valid custom element name
document.createElement('tabs') instanceof HTMLUnknownElement === true

// "x-tabs" is a valid custom element name
document.createElement('x-tabs') instanceof HTMLElement === true
```

所以我们可以在页面中先声明使用自定义元素，其后再调用 `customElements.define` 注册自定义元素，在自定义元素注册完成后，将页面中已声明使用的升级（upgrades）为自定义元素，这个过程称之为元素升级（element upgrades）


要了解标记名称何时注册定义，可以使用 `window.customElements.whenDefined()`。
```
customElements.whenDefined('app-drawer').then(() => {
  console.log('app-drawer defined');
});
```

## 自定义元素的状态 (state)
自定义元素的状态包括：**`undefined`, `failed`, `uncustomized`, `custom`**，其中 **`uncustomized`, `custom`** 都表已定义 **`defined`**；对于类似 `div` 的内置元素状态始终为已定义 **`defined`**

在元素升级之前，处于未定义状态，我们可以过 `:defined` 伪类预设置未注册元素的样式，避免在获得定义时产生布局跳动或者 FOUC

> FOUC: Flash of Unstyled Content 文档样式短暂失效，指 HTML 页面在打开过程中，内容先于样式展示，导致页面样式在瞬间出现剧变，并且人眼可见

```
app-drawer:not(:defined) {
  /* Pre-style, give layout, replicate app-drawer's eventual styles, etc. */
  display: inline-block;
  height: 100vh;
  opacity: 0;
  transition: opacity 0.3s ease-in-out;
}
```

在 `<app-drawer>` 获得定义后，选择器 `app-drawer:not(:defined)` 不再匹配。


## 设置自定义元素的内容

### 通过 `innerHTML` 设置元素内容
自定义元素在 `connectedCallback` 响应中，使用 DOM API 来设置元素的内容，但这不是好的做法，不推荐使用，更好的方式是使用 Shadow DOM。
```
window.customElements.define('x-tag', class extends HTMLElement {
  connectedCallback() {
    this.innerHTML = '<b>Hello World!</b> Custom Element'
  }
})
```

### 通过 Shadow DOM 设置元素内容
> 关于 Shadow DOM 的详细介绍，参考：[]()

Shadow DOM 提供了一种方法，可让设置的元素以独立于页面其余部分的方式拥有和渲染 DOM 并设置其样式。
```
<x-tag>Custom Element</x-tag>

window.customElements.define('x-tag', class extends HTMLElement {
  constructor() {
    super()

    let shadowRoot = this.attachShadow({mode: 'open'})
    shadowRoot.innerHTML = `
    <b>Hello World!</b>
    <slot></slot>
    `
  }
})
```

在这个例子中，Shadow DOM 是通过 `innerHTML` 来创建 DOM 元素；我们还可以通过 `<template>` 元素来创建 DOM。
```
<x-tag>Custom Element</x-tag>
<template id="tmpl">
  <style>
  /*scoped styles*/
  </style>
  <b>Hello World!</b>
  <slot></slot>
</template>

window.customElements.define('x-tag', class extends HTMLElement {
  constructor() {
    super()

    let shadowRoot = this.attachShadow({mode: 'open'})
    let tmpl = document.querySelector('#tmpl')
    shadowRoot.appendChild(tmpl.content.cloneNode(true))
  }
})
```

## 属性 (property) 映射为特性 (attribute)
对 DOM 元素属性 (property) 修改，其值会以特性 (attribute) 的形式映射到 DOM，例如通过 JS 修改 DOM 元素的 id，hidden 属性：
```
div.id = 'my-id';
div.hidden = true;
```
DOM 节点会变成：
```
<div id="my-id" hidden>
```

## 参考链接
本文主要是阅读 **Custom Elements v1: Reusable Web Components** 后，理解整理而来。
英文原文：[Custom Elements v1: Reusable Web Components](https://developers.google.com/web/fundamentals/web-components/customelements)
中文翻译：[自定义元素 v1：可重用网络组件](https://developers.google.com/web/fundamentals/web-components/customelements?hl=zh-cn)
