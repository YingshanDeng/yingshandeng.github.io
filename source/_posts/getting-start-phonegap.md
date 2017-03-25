title: PhoneGap 入门踩坑
date: 2017-03-25 16:27:07
tags:
categories: PhoneGap
---


公司产品在线文章编辑器 [写得](https://d.wps.cn/home.html) 今年需要进行移动 APP 的开发，经过考察决定使用 PhoneGap，尽量重用原有的 web 代码，打包成移动 APP。在接下来一段时间将对这个框架进行一些学习。

<!-- more -->

## 简介
[PhoneGap](http://phonegap.com/) 是一个跨平台的移动 app 开发框架，可以把 html css js 写的页面打包成跨平台的可以安装的移动 app，并且可以调用原生的几乎所有的功能，比如摄像头，联系人，加速度等。

📌 PhoneGap 和 Cordova 的关系？
Cordova 是 PhoneGap 贡献给 Apache 后的开源项目，是从 PhoneGap 中抽离出的核心代码，是驱动 PhoneGap 的核心引擎。有点类似 Webkit 和 Google Chrome 的关系。
> [PhoneGap, Cordova, and what’s in a name?](http://phonegap.com/blog/2012/03/19/phonegap-cordova-and-whate28099s-in-a-name/)
PhoneGap is a distribution of Apache Cordova. You can think of Apache Cordova as the engine that powers PhoneGap, similar to how WebKit is the engine that powers Chrome or Safari.

## 搭建开发环境
1. 全局安装 `phonegap`
```
npm i -g phonegap
```
2. 常用命令
```
// 创建一个app
phonegap create path/to/my-app
// 添加 iOS 平台项目
phonegap platform add ios --save
// 更新 iOS 项目代码文件
phonegap prepare ios
// 启动浏览器调试服务
phonegap serve
// iOS 模拟器中运行
phonegap run ios --emulator
phonegap run ios --emulator --target='iPhone-SE, 10.2'
```
3. 在 iOS 模拟器中调试

虽然 PhoneGap 也提供了在浏览器中调试页面的命令 `phonegap serve` 但是在移动开发过程中，有些例如沙盒，ImagePicker 等的调试，就需要用模拟器来进行。除了通过如上 `phonegap run ios --emulator` 命令启模拟器之外，我们还可以用 Xcode 打开 `platforms/ios/` 目录中的 `xcworkspace` 然后运行跑出模拟器。

然后我们可以通过 Safari 调试工具对模拟器中的网页进行调试。

❗️有一个需要注意的技巧：点击 Xcode 中 run 跑出模拟器，如果我们的前端代码有问题，出错的话，就会在出错出中断执行，但是在 Xcode 中的调试输出和 Safari 的调试输出中都没有错误信息，所以我们有时候可能并不会发觉出错了，或者不知道哪里出了问题。解决这个问题比较简单，我们只需要在 Safari 调试页面中 reload 一下页面即可。

- 方法一：
通常我们都是使用 Sublime Text 或者 Actom 这类编辑器来编写代码。编辑后，我们需要 `phonegap prepare ios` 来更新 iOS 项目，然后在 Xcode 中重新运行。重复如此的步骤，显然不是太方便。
- 方法二：
我们知道在 PhoneGap 打包移动应用，是把网页代码放在应用中，通过 file 协议进行加载，这样我们每次修改了页面，就需要通过 `phonegap prepare ios` 来更新 iOS 项目，然后需要 Xcode 重新运行。那如果我们把网页文件的引用修改成请求我们本地的 server 这样我们每次修改了文件，就只需要 reload 模拟器中的页面就可以了。

①启动本地 server
```
const gulp = require('gulp')
const browserSync = require('browser-sync')

gulp.task('debug', () => {
	browserSync({
        port: 8082,
        logPrefix: 'WPS Writer',
        open: false,
        server: {
            baseDir: ['www'],
            directory: true
        }
    });
})
```
②网页文件引用路径修改
```
<script src="scripts/index.js"></script>
变成-->
<script src="http://localhost:8082/scripts/index.js"></script>
```





