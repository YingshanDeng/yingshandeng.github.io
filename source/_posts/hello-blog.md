title: 你好，博客
date: 2016-07-16 16:14:51
tags: hexo
categories: 博客
---
 本文仅简单记录在创建博客过程中相关的内容，方便日后重建博客，有关细节检索等。

> **TL;DR**
1、采用 [Hexo](https://hexo.io/zh-cn/) 作为博客框架
2、采用 [Next](http://theme-next.iissnan.com/) 作为博客主题
3、使用 [七牛云存储](http://www.qiniu.com/) 作为图床，存放图片
4、使用 [多说](http://duoshuo.com/) 作为评论系统
5、使用 [Swiftype](https://swiftype.com/) 作为站内搜索 [**已经失效🙄**]
6、使用 [leancloud](https://leancloud.cn/) 为文章阅读次数统计

<!-- more -->

### Hexo

1、命令
``` bash
# 创建文章
hexo new "My New Post"

# 启动服务器
hexo server

# 部署
hexo clean
hexo generate
hexo deploy
```
2、有关部署
本博客是托管在 Github 上 [Github Pages](https://pages.github.com/) 所以是通过 Git 的方式部署的。根据 Hexo 文档的介绍，需要先安装一个 package
```
npm install hexo-deployer-git --save
```
然后修改配置：
```
deploy:
  type: git
  repo: <repository url>
  branch: [branch]
```
3、自定义域名
- DNS: 添加两个 A records: 192.30.252.153  192.30.252.154
- CNAME 文件，这文件需要放在部署文件的根目录，即 `public` 目录下。由于每次部署都需要重新生成该目录文件，那么这个文件要放到什么位置呢？**这个文件需要放在 hexo folder/source文件夹根目录下**。

4、关于文章分类和标签
在其他系统中，分类和标签听起来很接近，但是在 Hexo 中两者有着明显的差别：分类具有顺序性和层次性，也就是说 Foo, Bar 不等于 Bar, Foo；而标签没有顺序和层次。
```
categories:
  - Diary
tags:
  - tag1
  - tag2
```

5、关于[创建文章](https://hexo.io/zh-cn/docs/writing.html)
```
hexo new [layout] <title>
```
通过以上命令既可以创建一篇文章，其中 `layout` 只可以指定文章布局，包括 `post`、`page` 和 `draft`（默认是 `post`），分别对应不同的保存路径，其中需要留意的是 `post` 和 `draft`。其中前者会将文章进行展示发布，而后者只是草稿，并不进行展示。那么我们可以在编辑文章时，处于 `draft` 状态，在编辑完成后，需要发布时，才置为 `post` 状态。具体操作参考如下命令：
```
# 创建一篇草稿
hexo new draft test-draft

# 通过 publish 命令将 draft 文章转换为 post 文章
hexo publish draft test-draft

# 预览时添加 --draft 参数用于查看草稿文章
hexo server --draft
```

### 第三方服务集成

有关第三方服务在 Next 文档中有详细的介绍: [第三方服务集成](http://theme-next.iissnan.com/third-party-services.html)

### 友情链接

[Next 作者博客](http://notes.iissnan.com/)