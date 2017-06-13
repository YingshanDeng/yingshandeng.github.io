title: Cordova 代码热更新
tags:
---

![](http://7vikhl.com1.z0.glb.clouddn.com/app-store.png)

基于 Cordova 框架能将网页应用 (js, html, css, 图片等) 打包成 App。当 App 在应用商店上架后，如何快速更新是我们需要考虑的问题。🤖
- 本地打包新版本 App 发布到应用商店，但这中发布流程耗费时间，尤其是 Apple Store
- 应用加载网络资源，这样 App 展示的内容就可以保证是最新的，但当应用断网时，应用就无法正常使用

我们能想的这两种方式都存在的很大的弊端，不适合实际应用！

<!-- more -->

## 插件 Cordova Hot Code Push
插件 **[Cordova Hot Code Push](https://github.com/nordnet/cordova-hot-code-push)** 正是针对 Cordava 应用如何快速更新问题而提供的解决方案，可以自动更新 web 相关的静态文件。
> 该插件提供了详细的 wiki 文档，请参考：[wiki 文档](https://github.com/nordnet/cordova-hot-code-push/wiki)

### App Store 支持么
> 近日，苹果App Store审核团队向一些开发者下最后通牒：2017年6月12日之前移除所有热更新相关代码、框架或SDK，并重新提交版本。如果不作调整，App可能会从App Store下架。

苹果应用商店已经禁止使用类似 JSPatch 等热修复的框架或者SDK，那么这个插件提供的代码热更新功能是否违法这一规定呢? 🤔
📌 答案是否定的！此插件提供的代码热更新是 web 静态文件，而苹果是允许这一做法的。但有两点值得注意：
- ① 不能明显告知用户有新版本可用，询问用户是否需要更新到最新代码。这一做法会使用户产生困惑，这种更新方式和通过 App Store 更新有何区别。所以正确的做法是，在应用启动的时候，下载和安装热更新代码；或者在某个时机下载热更新代码，在应用下次打开时进行安装
- ② 如果通过此插件进行代码热更新后，应用功能发生巨大变化，譬如原来是一个计算器应用，代码热更新后，变成了一个音乐播放器，这种欺骗用户的做法也是会被苹果拒绝的

### 添加插件到项目中
1、下载插件
*要求 Cordova 版本 5.0+*
```
cordova plugin add cordova-hot-code-push-plugin --save
```
2、下载插件的命令行工具 [cordova-hot-code-push-cli](https://github.com/nordnet/cordova-hot-code-push-cli)
```
npm install -g cordova-hot-code-push-cli
```
该命令行工具可帮助我们自动生成配置文件 `chcp.json` 和 `chcp.manifest`，同时还提供了一些其他功能，详细可参考 README。

### 插件配置
TODO ...
`chcp.json` 和 `chcp.manifest`

config.xml

### 代码热更新原理
![](http://7vikhl.com1.z0.glb.clouddn.com/update-workflow.png)

- ① 用户打开应用
- ②
- ③

User opens your application.
Plugin get's initialized and it launches update loader in the background thread.
Update loader takes config-file from the config.xml and loads JSON from the specified url. Then it compares release version of the loaded config to the currently installed one. If they are different - we go to the next step.
Update loader uses content_url from the application config to load manifest file. He uses it to find out, what has changed since the last release.
Update loader downloads all updated/new files from the content_url.
If everything went well - it sends notification, that update is ready for installation.
Update installed, and user is redirected to the index page of your application.



TODO...
### Usage Limitations


### API 用法

##
```
```

##
```
```

## 参考链接
[]()
[]()