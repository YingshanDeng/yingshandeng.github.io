title: Cordova 开发环境 - Android 篇
date: 2017-07-03 22:14:19
tags:
categories: Cordova
---

![](http://cdn.objcer.com/android-banner.jpg)
<!-- more -->

## 安装 IDE (Android Studio)

安装 Android Studio, SDK, Android Virtual Device (AVD，安卓模拟器)
- 下载链接：[Android Studio](https://developer.android.com/studio/index.html?hl=zh-cn)
- 下载安装全程需要**翻墙**
- 安装 Android Studio 完成后，创建一个 HelloWorld 程序到模拟器中运行，这个过程中，按照提示进行下载安装即可

## 真机运行
手机通过 USB 连接电脑后，运行时，选择对应的设备即可运行，以下罗列几点真机运行中遇到的问题和技巧

### 解决：*DELETE_FAILED_INTERNAL_ERROR Error while Installing APK*
安卓真机（小米4C）通过 USB 连接电脑后，运行，可能会出现如下问题：
![](http://cdn.objcer.com/3B19ED97-CE3F-45A9-8518-C08D779F7FA3.png)
点击 OK 继续，会出现如下错误：
```
DELETE_FAILED_INTERNAL_ERROR
Error while Installing APKs
```
参考链接：[DELETE_FAILED_INTERNAL_ERROR Error while Installing APK](https://stackoverflow.com/questions/38892270/delete-failed-internal-error-error-while-installing-apk) 大致有两种解决方案：
- 操作步骤：Android Studio > Preferences >  Build, Execution, Deployment > Instant Run > **Uncheck : Enable Instant Run**（此处理方式会禁用 Instant Run 功能）
![](http://cdn.objcer.com/03E0830C-F5BB-4EC5-AC41-50A604A3E532.png)
- 由于调试设备是小米，所以还可以：设置 > 更多设置 > 开发者选项 > **去掉：启动 MIUI 优化** > 关闭并重启

### ADB 无线调试
Android ADB 提供了无线调试的功能
> [Android 调试桥](https://developer.android.com/studio/command-line/adb.html?hl=zh-cn#Enabling)

#### 解决：`zsh: command not found: adb`
解决步骤：
1️⃣ 执行如下命令往 .bash_profile 添加两个路径
```
echo "export PATH=\$PATH:/Users/${USER}/Library/Android/sdk/platform-tools/" >> ~/.bash_profile
echo "export PATH=\$PATH:/Users/${USER}/Library/Android/sdk/tools/" >> ~/.bash_profile
```
执行后 `.bash_profile` 文件：
```
export PATH=$PATH:/Users/YingshanDeng/Library/Android/sdk/platform-tools/
export PATH=$PATH:/Users/YingshanDeng/Library/Android/sdk/tools/
```
2️⃣ 执行 `source .bash_profile` 使其生效，此时我们在终端中输入 `adb` 会发现该命令生效了
3️⃣ 由于我的终端使用的是 zsh，所以需要再处理一下：
```
# 打开 .zshrc
open .zshrc

# 在文件末尾添加
source ~/.bash_profile

# 保存，重启终端即可
```

#### AndroidWiFiADB 插件
直接使用还是不够方便，推荐使用 **AndroidWiFiADB 插件**，项目地址：[AndroidWiFiADB](https://github.com/pedrovgs/AndroidWiFiADB)

安装方式：
- Preferences -> Settings -> Plugins-> Browse Repositories
- 输入: Android WiFi ADB 进行搜索，安装后重启

如何使用：
- 先将手机通过 USB 连接电脑，而且保证手机和电脑连接同一 WiFi，点击 AndroidWiFiADB 插件按键，会在右下角弹气泡提示是否连接成功，连接成功后即可断开 USB 连接线，享受无线提示
- 在使用过程中，要确保手机和电脑连接的同一 WiFi
- 可以通过 AndroidWiFiADB 设备面板查看设备及设置连接状态

![](http://cdn.objcer.com/36CDAE13-BEBC-42D2-9226-A53DF0CEE4B1.png)

## Chrome 远程调试模拟器/真机
> [远程调试 Android 设备使用入门](https://developers.google.com/web/tools/chrome-devtools/remote-debugging/?hl=zh-cn)

操作步骤：
- 随便在某个页面打开调试页面，如下图找到 **Remote devices**
![](http://cdn.objcer.com/8FA1EB72-12AE-4984-AA64-2C7022B1F00E.png)
- 确保已勾选 **Discover USB devices**
![](http://cdn.objcer.com/B378E60C-01E5-43AE-8EA1-02999501F241.png)
- 如下图找到对应的模拟器或者真机，点击 **Inspect** 按键就可以打开调试界面
![](http://cdn.objcer.com/917006D3-5881-4A6F-814F-0B3BB54383D7.png)

## 其他问题记录
1、Android Studio 导入项目卡在 *Building gradle project info* 的解决方法：
- [修改项目中gradle-wrapper.properties文件中的distributionUrl](http://www.jianshu.com/p/1311562bbfd4)
- 从一个能打开项目根目录拷贝 `gradle` 文件夹覆盖当前项目根目录中的 `gradle` 文件夹
