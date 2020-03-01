title: iOS 7 UIKit Dynamic
date: 2015-02-17 16:00:21
tags: iOS 7
categories: iOS
---

> iOS 7 中新增的 UIKit Dynamic 特性，为 UIKit 引入了2D物理引擎，通过它，可以更逼真、高效的实现对现实世界的仿真，同时也是你实现炫酷动画效果的利器。

<!--more-->

## Animators, Behaviors, and Dynamic Items

UIKit Dynamic 主要包含三个部分：

- **Dynamic item** -- 描述为一个力学物体（view）。但凡实现了 `UIDynamicItem` 协议的类对象都是 dynamic item， 这个协议中包含了一个 dynamic item 所必须的三个属性：`bounds` , `center`， `transform`

> **Apple 文档**：A dynamic item is any iOS or custom object that conforms to the UIDynamicItem protocol. The **UIView** and **UICollectionViewLayoutAttributes** classes implement this protocol starting in iOS 7.0. You can use a custom object as a dynamic item for such purposes as reacting to rotation or position changes computed by a dynamic animator—an instance of the UIDynamicAnimator class.

- **Behavior** -- 描述为一个力学行为，可以影响 dynamic item 的三个属性 `bounds` , `center`， `transform` ，例如：重力，碰撞等

> **Note** : UIDynamicBehavior：仿真行为，是动力学行为的父类，基本的动力学行为类UIGravityBehavior、UICollisionBehavior、UIAttachmentBehavior、UISnapBehavior、UIPushBehavior以及UIDynamicItemBehavior均继承自该父类

- **Animator** -- 描述为力学行为的执行者，是 Behavior 的容器，添加到其中的 Behavior 都会被 Animator 执行对 dynamic item 发挥作用。


**Note ：**

0、animator 中包含着 behavior，behavior 是作用到 dynamic item 上的，但是， 对于 dynamic item （通常就是些 view 视图）是不知道有哪些 behavior 会作用其上。
	譬如说：你对一个 view 添加了一个重力，然后 remove 掉这个 view，即使这个 view 已经不可见了，但是这个 gravity behavior 还是会对这个 view 进行 retain 然后执行重力效果。
	那么这样就必须手动控制，在 remove view 的时候，同时 dealloc behavior。
	同样的， behavior 和 animator 之间也是如此（do not know each other）。

1、物理单位

- 距离单位：现实世界以米为基本单位，在 UIKit 距离单位是点（point）。

> **Note** :  物体的重量 （mass）等于其面积（suqare area）乘以密度（density）。在 UIKit 中并没有为这个重量命名，我们暂且以 `UIKit kilogram`

- 时间单位：秒
- 力学单位：在现实世界中是以牛顿为基本单位，一牛顿的力可以让一公斤的物体获得 1 m/s^2 的加速度。而在 UIKit 中，一牛顿的力可以让 一 UIKit kilogram 的物体获得 100 p/s^2 的加速度。

>  **Note** :  在 UIKit 中，重力 g = 1000 p/s^2。当然默认情况下 UIKit 中是不会有重力存在的，你必须手动添加。


## Built-in Behavior (动力学行为)

 0.UIGravityBehavior：重力行为
赋予一个 dynamic item 一个类似现实世界的重力行为。其中有两个比较重要的属性。
``` objc
// The default value for the gravity vector is (0.0, 1.0)
// The acceleration for a dynamic item subject to a (0.0, 1.0) gravity vector is downwards at 1000 points per second².
// 重力方向（用向量表示）
@property (readwrite, nonatomic) CGVector gravityDirection;
// 重力大小
@property (readwrite, nonatomic) CGFloat magnitude;
// 默认情况下，重力方向向下，大小是 10 牛顿，也就是 1000 p/s^2
```
> - 在 UIKit 中是没有地面的，如果一个 dynamic item 接受的重力作用，那么在没有其他作用力阻碍下，它是会掉离屏幕的。
> - 一个 animator 只能接收一个 gravity behavior

 1.UICollisionBehavior：碰撞行为
UICollisionBehavior 会使 dynamic item 之间指定的边界范围内会发生碰撞，那么就需要指定边界范围。其中提供了几个API接口，常用到的是让 ReferenceView（参考视图）的边框范围为边界。

``` objc
// 设置 ReferenceView 边框为为边界
- (void)setTranslatesReferenceBoundsIntoBoundaryWithInsets:(UIEdgeInsets)insets;
// 指定一个贝塞尔路径为边界
- (void)addBoundaryWithIdentifier:(id <NSCopying>)identifier forPath:(UIBezierPath*)bezierPath;
// 指定两点之间的线段为边界
- (void)addBoundaryWithIdentifier:(id <NSCopying>)identifier fromPoint:(CGPoint)p1 toPoint:(CGPoint)p2;
```

`@protocol UICollisionBehaviorDelegate`

碰撞行为还有一个委托协议，其中接口方法可以让你在发生碰撞的过程中做一些自定义处理，例如，碰撞后发出声音等等。

 2.UISnapBehavior：吸附行为

吸附行为是将 dynamic item 通过一个类似弹簧效果的动画过程 ，在动作结尾有一个摇摆特效，移动到指定的位置（view center）
`- (instancetype)initWithItem:(id<UIDynamicItem>)item snapToPoint:(CGPoint)point`

其中只有一个属性
``` objc
// damping value from 0.0 to 1.0. 0.0 is the least oscillation.
@property (nonatomic, assign) CGFloat damping;
```

`damping` 表示振幅，是指在 snap 动作行为最后的摇摆效果的振幅大小。默认情况下是 0.5.（取值范围是 0.0 -- 1.0）

> **Note** :   如果想 snap 一个 dynamic item 到一个新的位置，那么只能 remove 原来的 snap behavior （如果有的话），然后再 add 一个新的 snap  behavior。

 3.UIAttachmentBehavior：附着行为

附着行为描述一个 dynamic item 与一个锚点（anchorPoint）或者另一个dynamic item 相连接的情况。
> **Note : ** 这里的锚点和坐标系统中的锚点不同，坐标系统中的锚点范围是 （0.0， 0.0）--（1.0， 1.0），默认情况下是 （0.5， 0.5）。而这里的锚点只是附着行为（两端中）一端的固定点坐标。

附着行为描述的是两点之间的连接情况，可以模拟**刚性**或者**弹性**连接

在多个物体间设定多个UIAttachmentBehavior，可以模拟多物体连接

**属性 ：**
- attachedBehaviorType：连接类型（连接到锚点或dynamic item）
``` objc
typedef NS_ENUM(NSInteger, UIAttachmentBehaviorType) {
    UIAttachmentBehaviorTypeItems,
    UIAttachmentBehaviorTypeAnchor
} NS_ENUM_AVAILABLE_IOS(7_0);
```

- anchorPoint：连接锚点坐标
- length：附着连接两端的距离

> 只要设置了以下两个属性，即为弹性连接

- damping：振幅大小
- frequency：振动频率

**Note ：** 对于 `UIAttachmentBehavior` 行为，如果单纯只是在 dynamic item 和锚点或者另一个 dynamic item 之间只有一个连接，那么有时候可能是不稳定的。那么为了避免这种情况，那么我们可以多加几个附着行为，也即是多加几个连接。

例如：假如有一张照片想固定在墙上，只是在左下角钉一个钉子（相当于附着一个点），那么在重力效果下，就会出现翻转，如下图（1）和（2）；那么为了这张照片可以固定，那么至少钉两个钉子（相当于钉附着两个点），如下图（3）钉了四个钉子。

![图片钉墙](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/photo.png)

代码如下：

``` objc
 UIView *box = [[UIView alloc] init];
    box.layer.bounds = CGRectMake(0, 0, 50, 50);
    box.layer.position = CGPointMake(200, 100);
    box.backgroundColor = [UIColor blueColor];
    [self.view addSubview:box];

    CGRect bounds = box.bounds;
    CGFloat width = CGRectGetWidth(bounds);
    CGFloat height = CGRectGetHeight(bounds);

    CGFloat offsetWidth = width/2;
    CGFloat offsetHeight = height/2;

    UIOffset offsetUL = UIOffsetMake(-offsetWidth, -offsetHeight);
    UIOffset offsetUR = UIOffsetMake( offsetWidth, -offsetHeight);
    UIOffset offsetLL = UIOffsetMake(-offsetWidth, offsetHeight);
    UIOffset offsetLR = UIOffsetMake( offsetWidth, offsetHeight);

    CGFloat anchorWidth = width/2;
    CGFloat anchorHeight = height/2;

    CGPoint anchorUL = CGPointMake(box.center.x - anchorWidth,
                                   box.center.y - anchorHeight);
    CGPoint anchorUR = CGPointMake(box.center.x + anchorWidth,
                                   box.center.y - anchorHeight);
    CGPoint anchorLL = CGPointMake(box.center.x - anchorWidth,
                                   box.center.y + anchorHeight);
    CGPoint anchorLR = CGPointMake(box.center.x + anchorWidth,
                                   box.center.y + anchorHeight);


    UIDynamicAnimator *animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];

    [animator addBehavior:[[UIAttachmentBehavior alloc] initWithItem:box offsetFromCenter:offsetUL  attachedToAnchor:anchorUL]];
    [animator addBehavior:[[UIAttachmentBehavior alloc] initWithItem:box offsetFromCenter:offsetUR attachedToAnchor:anchorUR]];
    [animator addBehavior:[[UIAttachmentBehavior alloc] initWithItem:box offsetFromCenter:offsetLL attachedToAnchor:anchorLL]];
    [animator addBehavior:[[UIAttachmentBehavior alloc] initWithItem:box offsetFromCenter:offsetLR attachedToAnchor:anchorLR]];

    [animator addBehavior:[[UIGravityBehavior alloc] initWithItems:@[box]]];

    self.animator = animator;
```

> **Note ：**
> - 1、 你可以注释掉几个 attachment behavior 看看效果。
> - 2、 对于两个 dynamic item 之间只有一个 attachment behavior，那么在添加了其他作用力的时候可能会出现旋转等 ‘副作用’

 4.UIPushBehavior：推行为

推行为（推力）可以为一个视图施加一个作用力，该力可以是持续的，也可以是一次性的，可以设置力的大小，方向和作用点等信息。

**属性：**
- mode：推动类型（一次性或是持续推力）

``` objc
typedef NS_ENUM(NSInteger, UIPushBehaviorMode) {
    UIPushBehaviorModeContinuous,
    UIPushBehaviorModeInstantaneous
} NS_ENUM_AVAILABLE_IOS(7_0);
```

- active：是否激活，如果是一次性推，需要激活
- angle：推动角度
- magnitude：推动力量
- pushDirection ：推力方向

> **Note : **
>  - 在 UIKit 中，一牛顿的力可以让一单位重量（面积：100p * 100p 乘以 密度为 1.0）的物体获得 100 p/s^2 的加速度。
>  - instantaneous push behavior （一次性推力）：**这个推力只会作用一秒钟**。 倒不如称之为一秒钟推力（O(∩_∩)O哈哈~）。作用后 `active` 属性会被置为 `NO` ，可以重置为 `YES` 来恢复这个推力的作用。
>  - continuous push behavior （持续推力）：这个推力会一直作用该 dynamic item 直到 remove 这个推力。

 5.UIDynamicItemBehavior：动力学元素行为

这个是一个可以自定义的作用力行为，通过设置运动力学元素参与物理仿真过程中的参数，如：弹性系数、摩擦系数、密度、阻力、角阻力以及是否允许旋转等来自定义一个作用力。

其中的属性包括：
``` objc
// 摩擦系数，范围是（0，1）
// Usually between 0 (inelastic) and 1 (collide elastically)
@property (readwrite, nonatomic) CGFloat elasticity;
// 摩擦系数
@property (readwrite, nonatomic) CGFloat friction;
// 密度，默认是 1
@property (readwrite, nonatomic) CGFloat density;
// 决定线性移动的阻力大小，与摩擦系数不同，摩擦系数只作用于滑动运动
@property (readwrite, nonatomic) CGFloat resistance;
// 角阻力，旋转时的阻力大小
@property (readwrite, nonatomic) CGFloat angularResistance;
// 是否允许旋转
@property (readwrite, nonatomic) BOOL allowsRotation;
```

> **Note :**  对于 friction 和 resistance : Friction influences items as they slide against other items in a straight line. Resistance influences items any time they move in a straight line. An item with low resistance but high friction still moves easily when pulled by an attachment. Angular resistance influences items any time they rotate. You can use the allowsRotation property to prevent rotation entirely.

最后：

> 所有的UIDynamicBehavior都是可以独立作用，同时也遵守力的合成。也就是说，组合使用行为可以实现一些较复杂的效果


## Apple官方Demo

[Demo 链接地址](https://developer.apple.com/library/ios/samplecode/DynamicsCatalog/Introduction/Intro.html)