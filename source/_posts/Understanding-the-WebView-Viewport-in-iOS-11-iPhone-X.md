title: 剖析 iOS 11 网页适配问题
date: 2017-09-21 15:55:40
tags: ['viewport-fit', 'iOS 11']
categories: CSS
---

北京时间 9 月 12 日凌晨，苹果在乔布斯剧院发布了 iPhone X。iPhone X 正面的全面屏上方有一条刘海，对于如何适配 iPhone X，苹果的 [Human Interface Guidelines](https://developer.apple.com/ios/human-interface-guidelines/overview/iphone-x/) 文档已经给出详细的说明。

苹果对于 iPhone X 的设计布局意见如下：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/iphone-x-safe-area.png)
核心内容应该处于 **Safe area** 确保不会被设备圆角(corners)，传感器外壳(sensor housing，齐刘海) 以及底部的 Home Indicator 遮挡。

本文将剖析两则在  iPhone X 异形屏和 iOS 11 网页适配中遇到的问题及解决方案。
<!-- more -->

## iPhone X Safari 横屏显示左右白边问题
iPhone X Safari 在横屏状态下，网页左右两侧可能会出现白边，如下：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/424F6040-3325-4AD5-8683-6D55AE4D9BEA.png)

因为 iPhone X 会将网页内容显示在 Safe area 导致的，解决这一问题，我们需要将背景颜色填充整个屏幕区域，而网页内容处于 Safe area。
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/47C98605-A351-4903-BDC7-C1E6878D0485.png)

### 解决方案一：`background-color`
如果网页设置了一个背景颜色，那么最简单解决方案是，在 `body` 节点设置 `background-color`，使背景颜色填充整个屏幕，从而解决横屏显示左右白边的问题

### 解决方案二：`viewport-fit` + `safe-area-inset-*`
在 iOS 11 中苹果为 Web 新增两个内容 `viewport-fit` + `safe-area-inset-*`
> ❗️注意：~~`viewport-fit` 和 `safe-area-inset-*` 只对于 WKWebView 有效，在 UIWebView 中无效~~

**纠正说明：**感谢 @dickeylth 和 @asiawang 二位的提醒说明，以上：~~`viewport-fit` 和 `safe-area-inset-*` 只对于 WKWebView 有效，在 UIWebView 中无效~~ 说法不对；应该是：**viewport-fit 和 safe-area-inset-* 是 iOS 11 才新增的内容，对于 iOS11 中的 WKWebView 和 UIWebView 都有效；但是对于 iOS10 及以下就不生效了**


1️⃣、`viewport-fit` 用于设置网页在可视窗口的布局方式
> 文档：[CSS Round Display Spec](https://drafts.csswg.org/css-round-display/#viewport-fit-descriptor)

这个属性可设置为：
- `contain`: The viewport should fully contain the web content. 可视窗口完全包含网页内容
- `cover`: The web content should fully cover the viewport.  网页内容完全覆盖可视窗口
- `auto`: The default value, `contain`

对于 iPhone X 之前的 iPhone 设备屏幕为规则的矩形，网页内容都可以完整展示；但是对于 iPhone X 是异形屏幕，通过 `viewport-fit` 可以设置网页内容在可视窗口中的显示规则，通过下图可以辅助理解
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/viewport-fit.png)

我们知道默认情况下 `viewport-fit=auto` 即为 `viewport-fit=contain`，在 iPhone X 中，相当于网页内容展示在 Safe area，这样也就是横屏显示时出现白边的原因了，所以我们可以设置 `viewport-fit=cover` 即可解决问题
```
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
```

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/Screen-Shot-2017-09-14-at-14.19.31.png)

注意到应用 `viewport-fit=cover` 之后，网页右上角，menu 按键和 in 按键被圆角，传感器外壳（齐刘海）裁剪掉了 😲

2️⃣、`safe-area-inset-*`
**在设置 `viewport-fit=cover` 之后**，Web 中会新增四个常量：
- safe-area-inset-top
- safe-area-inset-right
- safe-area-inset-left
- safe-area-inset-bottom

分别表示 Safe area 和可视窗口 viewport 顶部，右边，左边，底部的间距，可以用于设置 `margin`, `padding`, 或者绝对定位时 `left`, `top`
> ❗️注意：在横屏和竖屏状态下，`safe-area-inset-*` 的值不同

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/0C3B9577-094E-411C-9ED3-AA481FA3A50A.png)

为了解决应用 `viewport-fit=cover` 之后，有些显示内容被裁剪的问题，我们可以通过添加边距使得网页主要内容处于 Safe area 中不被裁剪，解决方式如下：
```
padding: constant(safe-area-inset-top) constant(safe-area-inset-right) constant(safe-area-inset-bottom) constant(safe-area-inset-left);
```

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/Screen-Shot-2017-09-14-at-14.07.11-1.png)

## iOS 11 WebView 中状态栏问题
**问题描述：**
在 iOS 11 由于 safe area，状态栏的表现有点不同。如果页面顶部有一个 header bar，设置 `position: fixed; top: 0px;` 那么页面滚动过程中，会出现如下问题：
![iPhone 6S](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/iPhone%206s.gif)
![iPhone X](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/iPhone%20X.gif)

header bar 没有固定在最顶部，上下滚动过程中，我们可以看到网页内容从 status bar 下方穿过
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/status-bar-problem.png)

**重现问题：**
创建一个 iOS 项目，视图中添加一个 `WKWebView`，加载百度首页
```
- (void)viewDidLoad {
  [super viewDidLoad];
  // Do any additional setup after loading the view, typically from a nib.
  WKWebView *webView = [[WKWebView alloc] initWithFrame:self.view.bounds];
  [webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"https://www.baidu.com"]]];
  [self.view addSubview:webView];
}
```

**剖析问题：**
1、这个问题同样也是由于 safe area 导致，虽然 header bar 位置信息为 `position: fixed; top: 0px;`，但这个位置也是相对于 safe area 而言的，所以看到 header bar 并没有位于屏幕最顶部。👉 在 viewport meta 标签，添加 `viewport-fit=cover` 即可解决此问题
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/2B17E02C-7C0C-4282-9369-F5E6D1C75836.png)

2、虽然 header bar 固定到屏幕最上方，但是很明显在 iPhone X 中却被圆角和齐刘海裁剪了内容，这怎么办呢？👉 为 header bar 添加 `padding-top` 即可解决此问题
```
header {
  /* ... */

  /* Status bar height on iOS 10 */
  padding-top: 20px;

  /* Status bar height on iOS 11+ */
  padding-top: constant(safe-area-inset-top);
}
```
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/FCF9D371-F2B5-44FE-9FFC-DC991E39FD2C.png)

## 更新内容
> **Webkit 文档** [Designing Websites for iPhone X](https://webkit.org/blog/7929/designing-websites-for-iphone-x/)

### CSS function `env()`
> The env() function shipped in iOS 11 with the name constant(). Beginning with Safari Technology Preview 41 and the iOS 11.2 beta, constant() has been removed and replaced with env(). You can use the CSS fallback mechanism to support both versions, if necessary, but should prefer env() going forward.

**Webkit 在 iOS 11 中新增 CSS function：`env()` 替代 `constant()`**，文档中推荐使用 `env()`，而 `constant()` 从 Safari Technology Preview 41 和 iOS 11.2 beta 开始会被废弃。`env()` 用法如同 `var()`，在不支持的 `env()` 的浏览器中，会自动忽略这一样式规则，不影响网页正常展示：
```
.post {
  padding: 12px;
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}
```

### CSS function `min()` 和 `max()`
在 iPhone X 设备设置网页边距的时候，可能会遇到这样的情形：我们通过 `env(safe-area-inset-left)` 和 `env(safe-area-inset-right)` 设置了页面展示左右边距：
```
.post {
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}
```

在横屏状态下显示正常，但是在竖屏状态下，常量 `safe-area-inset-left` 和 `safe-area-inset-right` 都为 0，所以会导致页面展示左右边距为 `0px`，如下图左，正常情况应该是如下图右，竖屏状态下页面左右也有边距。

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/css-funciton-min-max.png)

Webkit 从 Safari Technology Preview 41 和 iOS 11.2 beta 开始，新增 CSS function：`min()` 和 `max()`，这两个就可以解决这个问题了。
> Both functions take an arbitrary number of arguments and return the minimum or maximum. They can be used inside of calc(), or nested inside each other, and both functions allow calc()-like math inside of them.

```
@supports(padding: max(0px)) {
  .post {
    padding-left: max(12px, env(safe-area-inset-left));
    padding-right: max(12px, env(safe-area-inset-right));
  }
}
```

❗️此处需要使用 `@supports` 去检测浏览器是否支持 `min()` 和 `max()`

## 参考链接
[Human Interface Guidelines-iPhone X](https://developer.apple.com/ios/human-interface-guidelines/overview/iphone-x/)
[Removing the White Bars in Safari on iPhone X](http://stephenradford.me/removing-the-white-bars-in-safari-on-iphone-x/)
[Understanding the WebView Viewport in iOS 11](https://ayogo.com/blog/ios11-viewport/)