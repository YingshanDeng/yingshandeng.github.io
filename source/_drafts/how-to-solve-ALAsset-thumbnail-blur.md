title: 解决 ALAsset 缩略图模糊的问题
tags:
---

本文记录下，在自定义实现 iOS 相册 DEMO 中，解决使用 ALAsset 缩略图模糊的问题，如下图左，即为模糊的缩略图；如下图右，即为正常显示的缩略图。
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/1210059F-0531-4914-99FD-09FD86444B21.png)

相册 DEMO [项目地址](https://github.com/YingshanDeng/XDImagePicker)
<!-- more -->

## ALAssetsLibrary
ALAssetsLibrary 提供了我们对iOS设备中的相片、视频的访问，这个库在 iOS 9 之后已经废弃，但实际还是可用，官方建议使用 Photos Framework。

- AssetsLibrary: 代表整个设备中的资源库（照片库），通过 AssetsLibrary 可以获取和包括设备中的照片和视频;
- ALAssetsGroup: 映射照片库中的一个相册，通过 ALAssetsGroup 可以获取某个相册的信息，相册下的资源，同时也可以对某个相册添加资源;
- ALAsset: 映射照片库中的一个照片或视频，通过 ALAsset 可以获取某个照片或视频的详细信息，或者保存照片和视频；
- ALAssetRepresentation: ALAssetRepresentation 是对 ALAsset 的封装（但不是其子类），可以更方便地获取 ALAsset 中的资源信息，每个 ALAsset 都有至少有一个 ALAssetRepresentation 对象，可以通过 defaultRepresentation 获取

通过 ALAsset 属性 `thumbnail` 和 `aspectRatioThumbnail` 都可以获取到图片的缩略图。注意二者的区别：
- thumbnail: Returns a CGImage with a **square thumbnail** of the asset.（返回一个正方形的缩略图）
- aspectRatioThumbnail: Returns a CGImage with an **aspect ratio thumbnail** of the asset.（返回一个和图片宽高比相同的缩略图）

## 问题分析
```
CGImageRef thumbnailImageRef = [asset thumbnail];
self.imageView.image = [UIImage imageWithCGImage:thumbnailImageRef];
```
相册图片缩略图出现模糊，是由于使用了 `thumbnail` 属性，使用该属性的好处是，缩略图大小比较小，性能较好，页面在滚动过程中比较流畅；为了解决缩略图模糊的问题，我们就想到用 `aspectRatioThumbnail` 属性使用同图片宽高比相同的缩略图。

但是由于图片的宽高比都不一样，我们在一个固定宽高的 `UIImageView` 中展示时，就需要对其进行相应的处理。
```
// 使用 aspectRatioThumbnail 属性获取缩略图
CGImageRef thumbnailImageRef = [asset aspectRatioThumbnail];
self.imageView.image = [UIImage imageWithCGImage:thumbnailImageRef];

// 设置 UIImageView 中图片布局展示方式
[self.imageView setContentMode:UIViewContentModeScaleAspectFill];
[self.imageView setClipsToBounds:YES];
```
注意：UIImageView 的 contentMode 属性通常可以设定为如下常量：
```
typedef NS_ENUM(NSInteger, UIViewContentMode) {
    UIViewContentModeScaleToFill,
    // contents scaled to fit with fixed aspect. remainder is transparent
    UIViewContentModeScaleAspectFit,
	// contents scaled to fill with fixed aspect. some portion of content may be clipped.
    UIViewContentModeScaleAspectFill,
    UIViewContentModeRedraw,              // redraw on bounds change (calls -setNeedsDisplay)
    UIViewContentModeCenter,              // contents remain same size. positioned adjusted.
    UIViewContentModeTop,
    UIViewContentModeBottom,
    UIViewContentModeLeft,
    UIViewContentModeRight,
    UIViewContentModeTopLeft,
    UIViewContentModeTopRight,
    UIViewContentModeBottomLeft,
    UIViewContentModeBottomRight,
};
```
关于其中常用的设值区别如下：
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/contentMode.png)

## 问题完善
由于使用 `aspectRatioThumbnail` 属性获取图片的缩略图，在页面滚动过程中发现：快速滚动时会遇到一点卡顿的感觉，通过 Instrument 工具分析过程中发现：
```
CGImageRef thumbnailImageRef = [asset aspectRatioThumbnail];
```
此处的调用比较耗时

TODO ... 如何解决？

