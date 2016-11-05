title: 前端自动化构建及持续集成：Gulp + TeamCity
date: 2016-07-17 16:36:17

---

这篇文章主要讲述一下我在新项目中如何通过 Gulp  搭建项目的开发环境，自动化构建发布包以及通过 TeamCity 对项目进行可持续集成(CI).

> **TL;DR**
- Gulp 搭建开发环境：
	* http-server(BrowserSync)
	* 监听文件变化，对 JS 进行 babel 编译，对 CSS 进行 Post-CSS 处理等
- Gulp 自动化构建：
	* 构建区分：构建内网测试版本(Alpha)和构建正式环境版本(Release)
	* 任务包括：js babel，css post-css，文件拷贝，图片压缩，html 文件处理，文件合并，去除调试信息，html 文件最小化，gzip 压缩等等
- TeamCity 可持续集成
	* 监听代码仓库开发分支(dev)变化，进行自动化构建及发布到内网环境
	* 发布到正式环境

<!-- more -->


### Gulp


### TeamCity 持续集成(CI)

#### 何为持续集成？
> [百度百科](http://baike.baidu.com/view/5253255.htm)：持续集成是一种软件开发实践，即团队开发成员经常集成它们的工作，通过每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试）来验证，从而尽早地发现集成错误。

针对我们项目中的情况，CI 发挥了什么作用呢?

程序猿们每天都会对项目中特性进行修改，对BUG进行修复，那么完成后自然是要发布到内网环境进行测试的。那么如何最简化进行这个发布过程，提高工作效率呢？其中这个过程包括两个步骤：① 将前端代码进行构建，生成发布文件（静态文件） ② 将发布文件放到服务器。其中步骤①构建是需要花费一些时间的，然后步骤②也是需要后端哥哥辅助。

有了 CI, 我们只需要一部操作：将代码推送到远程分支，CI 监听分支变化，若有新的提交内容，就进行执行构建命令，构建成功后，执行发布命令，发布到内网环境。

![完美](http://7vikhl.com1.z0.glb.clouddn.com/perfect.gif)

目前 CI 工具比较火的包括 [TeamCity](https://www.jetbrains.com/teamcity/) [Jenkins](https://jenkins.io/index.html) [Travis CI](https://travis-ci.org/) (只适用于 Github 仓库) ... 然后通过 [这篇文章](http://jolestar.com/ci-teamcity-vs-jenkins/) 了解，最后选择了 TeamCity 作为 CI 系统。

#### 安装

1、TeamCity 支持 OSX, WINDOWS, LINUX 系统，选择一个喜欢的下载，这里选择 Linux 版本下载，安装在 Ubuntu. [下载链接](https://www.jetbrains.com/teamcity/download/)
2、安装包比较大，在下载的过程中，可以了解一下 TeamCity 系统中的一些概念 [介绍文档](https://confluence.jetbrains.com/display/TCD9/Continuous+Integration+with+TeamCity)
3、进行安装，[安装文档](https://confluence.jetbrains.com/display/TCD9/Installation+Quick+Start) 简单说一下在 Ubuntu 中安装步骤，大神忽略。
- 在 OSX 和 Ubuntu 中安装时，需要 JAVA 环境，要求 `since TeamCity 9.1 JDK 1.8, is recommended`（TeamCity 其实是一个 J2EE 应用）。Ubuntu 中安装 JAVA 环境参考：[参考链接](http://tecadmin.net/install-oracle-java-8-jdk-8-ubuntu-via-ppa/#)
- scp 将安装包拷贝到 Ubuntu 服务器
- 解压 `tar -zxvf TeamCity-9.1.6.tar.gz`
- 启动和关闭 在 `bin` 目录执行：
① To start the server and the default agent, use `runAll.sh start`
② To stop the server and the default agent, use `runAll.sh stop`

4、启动后，访问 `http://localhost:8111` 即可登录系统，初次登录，要进行初始化，一直点击下一步即可。
5、由于是构建前端环境，所以还要安装一下 `node` `npm` `bower`
- node 升级到最新版本
```
sudo npm cache clean -f
sudo npm install -g n
sudo n stable
```
- 由于 `npm` 比较慢，可以安装 `cnpm` [淘宝镜像](https://npm.taobao.org/)






