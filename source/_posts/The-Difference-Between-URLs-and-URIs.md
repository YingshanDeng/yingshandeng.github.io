title: 'URI, URL, URN 的区别'
date: 2017-06-04 16:43:26
tags: [URI, URL, URN]
categories: 编程
---


- URI: Uniform Resource Identifier (统一资源标识符)
- URL: Uniform Resource Locator (统一资源定位符)
- URN: Uniform Resource Name (统一资源名称)

<!-- more -->

## URI, URL, URN

**URI, URL, URN** 三者关系图:
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/URI-vs-URL.png)

📌 关系：
- URI 是一个紧凑的字符串用来标示抽象或物理资源
- URL 相对于 URI，除了确定一个资源，还提供一种定位该资源的主要访问机制或者网络地址 (“access mechanism” or “network location”) 例如：http:// or ftp://
- URI 包括 URL 和 URN，每一个 URL 都是 URI
- URN 是唯一标识的一部分，就是一个特殊的名字

![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/800px-URI_Euler_Diagram_no_lone_URIs.svg.png)
> *[维基百科]:* URI 可被视为定位符（URL），名称（URN）或两者兼备。统一资源名（URN）如同一个人的名称，而统一资源定位符（URL）代表一个人的住址。换言之，URN 定义某事物的身份，而 URL 提供查找该事物的方法。

举例说明：
- http://www.wikipedia.org/
	这个就是 URL，标识一个特定的互联网资源，并且可以通过 http 协议访问该网络资源
- urn:isbn:0-395-36341-1
	这个就是 URN，指定标识系统（即国际标准书号ISBN）和某资源在该系统中的唯一表示的URI。它可以允许人们在不指出其位置和获得方式的情况下谈论这本书

## 参考链接
[The Difference Between URLs and URIs](https://danielmiessler.com/study/url-uri/#gs.taQ4gpU)
[统一资源标志符](https://zh.wikipedia.org/zh-cn/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6)