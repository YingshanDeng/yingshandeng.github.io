title: iOS Hamburger Button动画效果两则
date: 2015-02-22 22:07:00
tags: iOS动画
categories: iOS
---

> **Hamburger Button** ：三个短线组成标识的button，button 点击后显示更多的内容。这篇文章将呈现两则 Hamburger Button 的动画效果。

<!--more-->

先看看效果：

![hambuger buttons](http://7vikhl.com1.z0.glb.clouddn.com/hamburger-btn.gif)

[Demo 下载地址](https://github.com/YingshanDeng/HamburgerButtonAnimation)

> 说明：下面只对第一则效果进行详细介绍。

## 初始化

对于这三条短线，在 `CAShapeLayer` 上绘制而成。

``` objc
/**
 *  初始化
 */
- (void)commonInit
{
    // 创建三条断线
    self.top = [CAShapeLayer layer];
    self.middle =[CAShapeLayer layer];
    self.bottom = [CAShapeLayer layer];

    _showMenu = YES;

    // 短线就是一个直线路径
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathMoveToPoint(path, nil, 0, 0);
    CGPathAddLineToPoint(path, nil, HamburgerButtonWidth, 0);

    for (CAShapeLayer *shapeLayer in @[self.top, self.middle, self.bottom])
    {
        shapeLayer.path = path;
        shapeLayer.lineWidth = 2.0f;
        shapeLayer.strokeColor = HamburgerButtonLineColor.CGColor;

        // 禁止隐式动画
        shapeLayer.actions = @{
                               @"transform": [NSNull null],
                               @"position": [NSNull null]
                               };
        // 创建一个已经 stoke 的路径用于获取 bound
        CGPathRef strokePath = CGPathCreateCopyByStrokingPath(path, nil, shapeLayer.lineWidth, kCGLineCapButt, kCGLineJoinMiter, shapeLayer.miterLimit);
        shapeLayer.bounds = CGPathGetBoundingBox(strokePath);

        [self.layer addSublayer:shapeLayer];
    }
    // 设置每一个短线的位置
    self.top.position = CGPointMake(HamburgerButtonWidth / 2, topYPosition);
    self.middle.position = CGPointMake(HamburgerButtonWidth / 2, middleYPosition);
    self.bottom.position = CGPointMake(HamburgerButtonWidth / 2, bottomYPosition);
}

```

其中有一点需要注意的是，要禁止 layer 的隐式动画。 [相关链接参考](http://stackoverflow.com/questions/2244147/disabling-implicit-animations-in-calayer-setneedsdisplayinrect)

## 动画过程

![动画始终状态](http://7vikhl.com1.z0.glb.clouddn.com/btn-change.png)

- top短线：旋转 -5/4 π 弧度，同时上移
- middle短线：翻转 π 弧度
- bottom短线：旋转 3/4 π 弧度，同时下移

![动画过程](http://7vikhl.com1.z0.glb.clouddn.com/hamburger-animate.gif)

三条短线的动画过程是同步的，所以都置于 `CATransaction` 中：

``` objc
[CATransaction begin];
[CATransaction setAnimationDuration:0.4f];
[CATransaction setAnimationTimingFunction:[CAMediaTimingFunction functionWithControlPoints:0.4 :0.0 :0.2 :1.0]];
// animations will go in here
......
[CATransaction commit];
```

中间短线的动画最简单，就是翻转 π 弧度，这里使用关键帧动画。

``` objc
// middle line
CAKeyframeAnimation *middleRotation = [CAKeyframeAnimation animationWithKeyPath:@"transform"];
middleRotation.values = [self rotationValuesFromTransform:self.middle.transform WithEndValue:_showMenu ? (-M_PI) : (M_PI)];
[self.middle addAnimation:middleRotation forKey:@"middleRotation"];
[self.middle setValue:middleRotation.values[middleRotation.values.count - 1] forKeyPath:middleRotation.keyPath];
self.middle.strokeEnd = _showMenu ? 1.0 : 0.85;
```

其中需要注意的是：在动画结束后要通过以下方法设置当前 layer 的最终状态，否则动画结束后，又会回到动画开始时的状态。

`- (void)setValue:(id)value forKeyPath:(NSString *)keyPath`

> **Note ：**有关动画的详细内容，参考 [Apple 文档](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004514-CH1-SW1)

第一条top短线和最后一条bottom短线的动画过程差不多，都需要两个动画：旋转和位移。

``` objc
// 根据是否显示 menu 来决定 stroke Start 的值
CGFloat strokeStartNewValue = _showMenu ? 0.0 : 0.3;
CGFloat positionPathControlPointY = bottomYPosition / 2;
// 竖直方向上的偏移
CGFloat verticalOffsetInRotatedState = 0.75;

// top line
CAKeyframeAnimation *topRotation = [CAKeyframeAnimation animationWithKeyPath:@"transform"];
// top 旋转角度: 开始是 旋转 - 5/4 pi , 结束是 旋转 5/4 pi  （逆时针方向是正，顺时针方向是负）
topRotation.values = [self rotationValuesFromTransform:self.top.transform WithEndValue:_showMenu ? (-M_PI - M_PI_4) : (M_PI + M_PI_4)];
topRotation.calculationMode = kCAAnimationCubic;
topRotation.keyTimes = @[@0.0, @0.33, @0.73, @1.0];
[self.top addAnimation:topRotation forKey:@"topRotation"];
[self.top setValue:topRotation.values[topRotation.values.count - 1] forKeyPath:topRotation.keyPath];

CAKeyframeAnimation *topPosition = [CAKeyframeAnimation animationWithKeyPath:@"position"];
CGPoint topPositionEndPoint = CGPointMake(HamburgerButtonWidth / 2, _showMenu ? topYPosition : (bottomYPosition + verticalOffsetInRotatedState));
// 位置沿着贝塞尔曲线路径进行移动
topPosition.path = [self quadBezierCurveFromStartPoint:self.top.position withToPoint:topPositionEndPoint withControlPoint:CGPointMake(HamburgerButtonWidth, positionPathControlPointY)].CGPath;
[self.top addAnimation:topPosition forKey:@"topPosition"];
[self.top setValue:[NSValue valueWithCGPoint:topPositionEndPoint] forKey:topPosition.keyPath];
self.top.strokeStart = strokeStartNewValue;
```

其中需要指出的是：
- `strokeStart` 和 `strokeEnd` ：指定路径绘制的起点和终点，范围是 `[0,1]`，那么这样就可以定制线条的长短变化。
- 箭头的形状 ：为了绘制这个箭头，我们是将 top 短线和 bottom 短线设置成  `90`  度。


## 用法

``` objc
CGRect screenRect = [[UIScreen mainScreen] bounds];

self.btnOne = [[HamburgerButtonOne alloc] initWithFrame:CGRectZero];
self.btnOne.transform = CGAffineTransformMakeScale(4.0, 4.0);
self.btnOne.center = CGPointMake(screenRect.size.width / 2, screenRect.size.height / 2 - 100);
[self.btnOne addTarget:self action:@selector(btnOntPressed:) forControlEvents:UIControlEventTouchUpInside];
[self.view addSubview:self.btnOne];

- (void)btnOntPressed:(id)sender
{
    self.btnOne.showMenu = !self.btnOne.showMenu;
}
```

``` objc
- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:CGRectMake(0, 0, HamburgerButtonWidth, HamburgerButtonHeight)];
    if (self)
    {
        [self commonInit];
    }
    return self;
}
```

说明：其中我们在 `initWithFrame:` 方法中设置了大小，所以我们使用的时候就直接给个 `CGRectZero`就可以了。


## 其他
**说明：以上两则hambuger button动画效果改自以下 swift 版本：**

* [Swift Hamburger Button - 1](https://github.com/fastred/HamburgerButton)
* [Swift Hamburger Button - 2](https://github.com/robb/hamburger-button)

**关于 hambuger button 的一些讨论：**

* [The Hamburger Menu-Icon Debate](http://www.theatlantic.com/product/archive/2014/08/the-hamburger-menu-debate/379145/)
* [Why and How to Avoid Hamburger Menus](https://lmjabreu.com/post/why-and-how-to-avoid-hamburger-menus/)

