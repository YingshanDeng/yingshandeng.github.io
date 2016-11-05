title: DNS Prefetching
date: 2016-09-25 17:39:24
tags: DNS Prefetching
categories: HTML
---

在浏览器访问一个域名的时候，都会经过一个域名解析的过程，也就是 DNS。域名解析需要消耗时间，如何减少解析等待时间，也是前端页面优化的一个课题。DNS 预加载 (DNS Prefetching) 作为现代浏览器的一大特性，可以减少用户的等待时间，提升用户体验，是Web页面性能优化的一个方式。

> 参考文章: [官方文档](https://www.chromium.org/developers/design-documents/dns-prefetching)

<!-- more -->

### DNS 解析方式

1、自动获取: 会把当前页面中所有带 `href` 属性的超链接进行 DNS 解析

> Chromium uses the "href" attribute of hyperlinks to find host names to prefetch

```
<a href="http://www.objcer.com">a</a>
```

2、手动获取

> However, some of those hyperlinks may be redirects, for example if the site is trying to count how many times the link is clicked.  In those situations, the "true" targeted domain is not necessarily discernible by examining the content of a web page, and so Chromium not able to prefetch the final targeted domain

但是，如果有些超链接做了重定向，那么在这时自动解析DNS就无法得到正确的 IP 地址，那么就应该手动设置，将实际要DNS解析的域名写到 link 标签的 `href` 属性中。

```
<link rel="dns-prefetch" href="//host_name_to_prefetch.com">
```

我们可以注意到百度页面 head 中就包含了如下一段 DNS 预加载的 link 标签：

![](http://7vikhl.com1.z0.glb.clouddn.com/baidu-dns-prefetching.png)

### DNS Prefetch Control

>- By default, Chromium does not prefetch host names in hyperlinks that appear in HTTPS pages.
- To allow webmasters to control whether DNS prefetch is enabled or disabled, Chromium includes a DNS Prefetch Control mechanism. It can be used to turn DNS prefetch on for HTTPS pages, or turn it off for HTTP pages.

超链接在 https 下是不会启用 DNS 预解析的，但是浏览器提供了一个开关来设置是否在 HTTPS 下启用预解析，是否在 HTTP 下关闭预解析。

```
<meta http-equiv="x-dns-prefetch-control" content="on">
<meta http-equiv="x-dns-prefetch-control" content="off">
```

### 其他

1、DNS prefetching is an attempt to resolve domain names before a user tries to follow a link. This is done using the computer's normal DNS resolution mechanism;

2、 Chromium currently employs 8 completely asynchronous worker threads to do nothing but perform DNS prefetch resolution.

### 参考文章

[参考链接1](http://www.jianshu.com/p/c3a14a853c79)
[参考链接2](https://segmentfault.com/a/1190000003944417)
