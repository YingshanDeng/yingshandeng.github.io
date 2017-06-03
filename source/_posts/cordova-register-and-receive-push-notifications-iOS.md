title: Cordova 远程推送 - iOS篇
date: 2017-06-04 01:03:12
tags: Push Notification
categories: Cordova
---

![](http://7vikhl.com1.z0.glb.clouddn.com/iOS-PUsh-Notifications.png)

<!-- more -->

## APNs
> APNs(*Apple Push Notification service*) [官方文档 Overview](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1)

### 推送原理
当 Provider 远程服务器需要向 App 推送一条消息时，并不是直接往 App 推送消息，而是推送苹果 APNs 服务器上，而苹果的 APNs 服务器再通过与设备建立长连接进而把消息推送到我们的设备上。
![](http://7vikhl.com1.z0.glb.clouddn.com/82BDFC5F-339D-4402-8719-5197D0246305.png)

由此：APNs 推送就划分成两部分：
- Provider-to-APNs connection (服务端交互)
- APNs-to-device connection (客户端交互)

### deviceToken
1、当一个 App 注册接收远程通知时，系统会发送请求到 APNs 服务器，APNs 服务器收到此请求会生成一个**独一无二**的 deviceToken，发送到对应请求的 App 上。然后 App 把此 deviceToken 发送给我们自己的服务器 Provider。
> ❗️一个 deviceToken 可以唯一标识某个设备上的某个 App，可以理解成：**deviceToken = device UUID + App Bundle ID**

![](http://7vikhl.com1.z0.glb.clouddn.com/Managing%20the%20device%20token.png)

2、Provider 给我们的设备推送通知的时候，必须包含此 deviceToken。
![](http://7vikhl.com1.z0.glb.clouddn.com/Remote%20notification%20path%20from%20provider%20to%20device.png)

### 推送消息 Payload
> [Creating the Remote Notification Payload](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html#//apple_ref/doc/uid/TP40008194-CH10-SW1)

1、Payload 消息大小
- For regular remote notifications, the maximum size is 4KB (4096 bytes)
- For Voice over Internet Protocol (VoIP) notifications, the maximum size is 5KB (5120 bytes)
- If you are using the legacy APNs binary interface to send notifications instead of an HTTP/2 request, the maximum payload size is 2KB (2048 bytes)

当 Payload 负载大小超过规定的负载大小时，APNs 会拒绝发送此消息。

2、Payload 消息格式
```
// 以下为 Payload 常用字段，详细请参考官方文档
{
    "aps" : {
        "alert"              :   {   // string or dictionary
            "title"          :   "string",
            "body"           :   "string"
        },
        "badge"              :    number,
        "sound"              :    "string"
    },
    "acme1" : [ "foo",  "bar" ]
}
```
- aps：推送消息必须有的 key
    - alert：推送消息包含此 key 值，系统就会根据用户的设置展示标准的推送信息
        - title：简短描述此调推送消息的目的
        - body：推送的内容
    - badge：在 app 图标上显示消息数量，缺少此 key ，消息数量就不会改变，消除标记时把此 key 对应的 value 设置为0
    - sound：设置推送声音的key值，系统默认提示声音对应的 value 值为 default
- acme: 以 `acme` 作为前缀，表示自定义数据

例如：
```
{
    "aps" : {
        "alert": "This is a notification",
        "badge": 1
    },
    "acme-data": [ "foo",  "bar" ]
}

{
    "aps" : {
        "alert": {
            "title": "This is Title",
            "body": "This is Body"
        },
        "badge": 1
    },
    "acme-data": [ "foo",  "bar" ]
}
```

3、当设备处于离线 offline 状态时，APNs 会为保留 Provider 所推送的最后一条通知，当设备转换为连网状态时，APNs则把其保留的最后一条通知推送给我们的设备；如果设备长时间离线，那么最后一条通知也有可能被丢弃。

### 总结
![](http://7vikhl.com1.z0.glb.clouddn.com/apns-app.png)

- ① App 向系统发起请求
- ② 系统向 APNs 发送注册远程推送请求
- ③ APNs 生成 deviceToken 并返回
- ④ App 将 deviceToken 发送给服务器 Provider
- ⑤ 从推送管理平台向 Provider 发出推送消息指令
- ⑥ Provider 推送消息给 APNs
- ⑦ APNs 推送消息给 App

## 安装插件
Cordova 项目要集成远程推送功能，需要用到 [phonegap-plugin-push](https://github.com/phonegap/phonegap-plugin-push) 这个插件，该插件提供了对多个平台的推送支持，当然包括 iOS 和 Android，下面以 iOS 项目为例，详细介绍整个安装过程。

> iOS 安装过程：[参考文档](https://github.com/phonegap/phonegap-plugin-push/blob/master/docs/INSTALLATION.md#ios-details)

❗️注意：
- 以下安装过程需要**翻墙**
- 该插件要求 cordova 版本至少 6.1.0，否则要么更新 cordova 或者下载时指定该插件版本号：
```
cordova plugin add phonegap-plugin-push@1.8.1
```
执行安装命令：
```
➜  wps-writer git:(feature/edit) cordova plugin add phonegap-plugin-push
Fetching plugin "phonegap-plugin-push@1.10.4" via npm
Installing "phonegap-plugin-push" for browser
Installing "phonegap-plugin-push" for ios
Failed to install 'phonegap-plugin-push':undefined
Error: CocoaPods was not found. Please install version 1.0.1 or greater from https://cocoapods.org/
```
提示我们要安装 CocoaPods，执行如下命令进行安装：
```
➜  wps-writer git:(feature/edit) ✗ sudo gem install cocoapods
ERROR:  Could not find a valid gem 'cocoapods' (>= 0), here is why:
          Unable to download data from https://ruby.taobao.org/ - Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://ruby.taobao.org/specs.4.8.gz)
```
根据错误提示我们知道，从淘宝 Ruby 的软件源中找不到 cocoapods，我们需要增加 gem 源
```
# 增加/删除 gem 源
gem sources --add https://rubygems.org/
gem sources --remove https://rubygems.org/
# 查看 gem 源
gem sources -l
```
安装完 CocoaPods 之后，重新安装插件：
```
➜  wps-writer git:(feature/edit) cordova plugin add phonegap-plugin-push
Fetching plugin "phonegap-plugin-push@1.10.4" via npm
Installing "phonegap-plugin-push" for browser
Installing "phonegap-plugin-push" for ios
Failed to install 'phonegap-plugin-push':Error: pod: Command failed with exit code 1
    at ChildProcess.whenDone (/Users/YingshanDeng/Documents/WebPS/wps-writer/platforms/ios/cordova/node_modules/cordova-common/src/superspawn.js:169:23)
    at emitTwo (events.js:100:13)
    at ChildProcess.emit (events.js:185:7)
    at maybeClose (internal/child_process.js:827:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:211:5)
Error: pod: Command failed with exit code 1
```
安装文档中也可以找到 [解决这个问题](https://github.com/phonegap/phonegap-plugin-push/blob/master/docs/INSTALLATION.md#common-cocoapod-installation-issues) 的介绍：
- 进入 `platforms/ios` iOS 项目根目录，执行 `pod repo update`
- 重新安装该插件

但是执行 `pod repo update` 实在是太慢了💩，不能忍。我们可以通过如下方法进行解决：
```
sudo rm -fr ~/.cocoapods/repos/master
pod setup
```
执行 `pod setup` 会 Specs 整个仓库 clone 到本地，这个仓库是是所有 Pods 的索引，就是一个容器，所有公开的 Pods 都在这里面。整个下载大小约 400M 所以需要点时间 🤣

下载完毕后，我们再进入 iOS 项目中执行 `pod repo update` 就很快了。以上操作都顺利完成后，重新安装该插件：
```
# 先移除后添加
cordova plugin rm phonegap-plugin-push
cordova plugin add phonegap-plugin-push
```
不出意外就安装成功了 ✌️。

总结一下其实就是两点：
- `sudo gem install cocoapods`
- `pod setup`

## 使用插件
> [API 文档](https://github.com/phonegap/phonegap-plugin-push/blob/master/docs/API.md#pushnotificationinitoptions)

插件在全局添加了一个 PushNotification 类。

### `PushNotification.init(options)`
推送初始化，通过传入配置信息，包含各个平台相应的设置，返回 `PushNotification` 实例
❗️注意：
- 必须在 `deviceready` 事件回调中调用 `PushNotification.init()`
- 每一次 App 启动，都需要调用 `PushNotification.init()`，因为 App 对应的推送服务 registration ID 可能发生变化（对于 iOS 即是 deviceToken）

```
 push = PushNotification.init({
    // android: {
    // },
    // browser: {
    // },
    ios: {
        alert: "true", // 弹出通知提示
        badge: "true", // 应用图标右上角数字标记
        sound: "true", // 通知提示音
        clearBadge: "true" // 应用打开后清除图标标记
    }
});
```

### 事件监听 `push.on(event, callback)`
```
// ① 注册通知成功回调，返回的 registrationId
// 对于 iOS，向 APNs 注册远程推送，返回的 deviceToken 即 registrationId
push.on('registration', (data) => {
    console.log('---registration---', data.registrationId)
});

// ② 接收到远程推送消息的回调
push.on('notification', (data) => {
    // data.message,
    // data.title,
    // data.count,
    // data.sound,
    // data.image,
    // data.additionalData
    console.log('---notification---', data)
});

// ③ 内部错误回调
push.on('error', (e) => {
    // e.message
    console.log('---notification error---', e.message);
});
```

### `PushNotification.hasPermission`
检查 App 是否授权允许推送通知
```
checkPermission: function () {
    return new Promise((resolve, reject) => {
        PushNotification.hasPermission(function(data) {
            // resolve(isEnabled)
            resolve(data.isEnabled);
        });
    });
}
```

### `get/setApplicationIconBadgeNumber`
获取/设置应用图标标记数字
```
setIconBadgeNumber(count) {
    return new Promise((resolve, reject) => {
        // resolve()
        push.setApplicationIconBadgeNumber(resolve, reject, count);
    });
},
getIconBadgeNumber() {
    return new Promise((resolve, reject) => {
        // resolve(count)
        push.getApplicationIconBadgeNumber(resolve, reject);
    });
}
```

### badge 自增问题 🤖
通常来说，应用在接收到一条推送消息通知的时候，图标标记 badge 就应该加一；当打开应用的时候，badge 清零。

我们知道通过设置 Payload 消息中的 `badge` 字段，可以设置应用接收到推送消息时，图标标记设置的数值（`badge` 字段为零即为清空图标标记数字）；而在 PushNotification 插件初始化时通过配置传入 `clearBadge` 可以设置打开应用时，图标标记清零。

但是能否做到接收到推送消息时，badge 显示数值自增一呢？就目前 PushNotification 插件来说，并不能做到。而对于微信或者 QQ 这类应用的图标 badge 数值提示的是未读消息数，其设置逻辑应该是通过服务端维护的。

对于图标标记的设置，参考了一些 App 的做法：
- 设置 `badge` 字段为 1
- 不展示图标标记（设置 `badge` 字段为 0 或者 PushNotification 插件初始化是 `badge` 设置为 `false`）

## 推送证书
具备接收远程推送的应用打包时需要使用推送证书，同样也区分开发证书和发布证书。

### 导出推送证书
登录 [苹果开发者中心](https://developer.apple.com) 在 `Certificates, Identifiers & Profiles -> Identifiers -> App IDs`，找到应用对应的 App ID，点击 `Edit` 对 Push Notifications 进行配置导出 certificate 证书。
![](http://7vikhl.com1.z0.glb.clouddn.com/CreateCertificate.png)

双击安装证书如下：
![](http://7vikhl.com1.z0.glb.clouddn.com/aps.png)

同时我们也需要更新下载 Provisioning Profile 文件。

### Xcode 真机调试推送配置
1、target -> Capabilities
![](http://7vikhl.com1.z0.glb.clouddn.com/capabilities.jpg)

2、target -> Build Settings -> Signings
![](http://7vikhl.com1.z0.glb.clouddn.com/build-settings.jpg)


## 参考链接
[iOS推送之远程推送（iOS Notification Of Remote Notification）](http://www.jianshu.com/p/4b947569a548)