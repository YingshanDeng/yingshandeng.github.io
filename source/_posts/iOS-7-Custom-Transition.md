title: iOS 7 Custom Transition
date: 2015-02-14 20:37:02
tags: iOS 7
categories: iOS
---

> iOS 7 中为 ViewControllers 间（Navigation Push / Pop  或者  Modal Present / Dismiss）的切换和交互增加了特性，你可以**自定义切换的动画效果**和**切换过程的交互动作**

<!-- more -->

## API 介绍
> 相关的内容都定义在： `UIVIewControllerTransitioning.h`  文件中

0、`@protocol UIViewControllerContextTransitioning`

这个协议的接口中定义了一些 ViewControllers 之间切换上下文（动画+交互）中所需的相关信息。

一般来说，The UIViewControllerContextTransitioning protocol can be adopted by **custom container controllers**。我们平时只需关注其中的方法，拿来用就好了。

例如：
``` objc
// 切换交互动作相关的方法
- (void)updateInteractiveTransition:(CGFloat)percentComplete;
- (void)finishInteractiveTransition;
- (void)cancelInteractiveTransition;
// 根据 key 值获取 toVC 和 fromVC
- (UIViewController *)viewControllerForKey:(NSString *)key;
// 获取 toVC 和 fromVC 的 Frame
- (CGRect)initialFrameForViewController:(UIViewController *)vc;
- (CGRect)finalFrameForViewController:(UIViewController *)vc;
......
```

1、 `@protocol UIViewControllerAnimatedTransitioning`

这个协议的接口负责切换过程的动画效果。一般的，我们可以通过创建一个（subclass of NSObject）类实现这个接口，然后通过该类对象来使用自定义的动画效果。

``` objc
// 动画时间
- (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext;
// 动画内容
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext;
// 以上两个方法必须实现，下面的方法可选。
@optional
// This is a convenience and if implemented will be invoked by the system when the transition context's completeTransition: method is invoked.
- (void)animationEnded:(BOOL) transitionCompleted;
```

2、 `UIViewControllerInteractiveTransitioning`

这个协议的接口负责切换过程的交互动作。这个接口比较简单，我们只需在 ：`- (void)startInteractiveTransition:(id <UIViewControllerContextTransitioning>)transitionContext;
` 方法中实现所需的切换交互动作即可。

当然其中还有两个可选实现方法：
``` objc
- (CGFloat)completionSpeed;
- (UIViewAnimationCurve)completionCurve;
```

> **Note**: 一般来说 view controller 间的动画切换都是通过某些手势来实现的。通常用到的时 **Pan** 手势。

3、 `UIPercentDrivenInteractiveTransition`

系统已经为我们实现了一个交互动作 -- 百分比切换动作，这个类实现了 `UIViewControllerInteractiveTransitioning` 接口，我们可以直接创建这个类对象进行使用（一般都是结合 Pan 手势使用）

其中比较重要的方法有：
``` objc
// 更新交互动作的完成比例
- (void)updateInteractiveTransition:(CGFloat)percentComplete;
// 取消交互动作
- (void)cancelInteractiveTransition;
// 完成交互动作
- (void)finishInteractiveTransition;
......
```

4、 `UIViewControllerTransitioningDelegate`

这个协议负责向系统返回切换的动画效果和切换的交互动作。我们在需要进行 Custom Transition 的 view controller 中实现这个接口，返回相应的对象即可。（一般来说，在  Modal Present / Dismiss 切换中实现以下协议）
``` objc
@optional
// 动画效果
- (id <UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source;

- (id <UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed;

// 交互动作
- (id <UIViewControllerInteractiveTransitioning>)interactionControllerForPresentation:(id <UIViewControllerAnimatedTransitioning>)animator;

- (id <UIViewControllerInteractiveTransitioning>)interactionControllerForDismissal:(id <UIViewControllerAnimatedTransitioning>)animator;

// iOS 8 新增
- (UIPresentationController *)presentationControllerForPresentedViewController:(UIViewController *)presented presentingViewController:(UIViewController *)presenting sourceViewController:(UIViewController *)source NS_AVAILABLE_IOS(8_0);

@end
```

5、`UINavigationControllerDelegate`

对于 Navigation Controller 中的（Push / Pop）切换，我们需要实现该协议，选择性实现以下的方法：
``` objc
// 交互动作
- (id <UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController
                          interactionControllerForAnimationController:(id <UIViewControllerAnimatedTransitioning>) animationController NS_AVAILABLE_IOS(7_0);
// 切换动画
- (id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC  NS_AVAILABLE_IOS(7_0);
```


> **题外话**：我们注意到以上 API 接口中，很多都是委托协议。那么有一点值得提示的是，我们在 Modal Present / Dismiss 时，当要进行 Dismiss 时候，苹果建议也是用委托的方式进行（此处不展开）。
>
>  [Apple 官方文档](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/ModalViewControllers/ModalViewControllers.html#//apple_ref/doc/uid/TP40007457-CH111-SW14)
>
>  When it comes time to dismiss a presented view controller, the preferred approach is to let the presenting view controller dismiss it. In other words, whenever possible, the same view controller that presented the view controller should also take responsibility for dismissing it. Although there are several techniques for notifying the presenting view controller that its presented view controller should be dismissed, the preferred technique is delegation.

## Demo

> **说明**：从 iOS 7 开始 Navigation Controller 附赠了一个从屏幕左侧边缘右滑返回的手势，但是仅限于屏幕左侧边缘，那么如何扩大到整个屏幕呢？当然，有了以上这些强有力的 API ，我们是可以实现的。

[demo下载链接](https://github.com/YingshanDeng/CustomTransitionDemo)


效果如下：

![demo效果](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/demo.gif)


1、基础代码（项目中包含两个VC：FirstViewController SecondViewController）
``` objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    FirstViewController *firstVC = [[FirstViewController alloc] init];
    UINavigationController *naviController = [[UINavigationController alloc] initWithRootViewController:firstVC];
    self.window.rootViewController = naviController;
    [self.window makeKeyAndVisible];
    return YES;
}
```
其中 FirstViewController 视图中有一个 btn 用于跳转到 SecondViewController
``` objc
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    [self.view setBackgroundColor:[UIColor whiteColor]];
    self.title = @"FirstVC";

    CGRect rect = [[UIScreen mainScreen] bounds];
    UIButton *btn = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    btn.layer.bounds = CGRectMake(0, 0, 100, 50);
    btn.layer.position = CGPointMake(rect.size.width / 2, rect.size.height / 2);
    [btn setTitle:@"Click -- Push" forState:UIControlStateNormal];
    [btn addTarget:self action:@selector(btnPressed:) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:btn];
}


- (void)btnPressed:(UIButton *)btn
{
    SecondViewController *secondVC = [[SecondViewController alloc] init];
    [self.navigationController pushViewController:secondVC animated:YES];
}

```

2、定制 Pan 手势，我们只在 Pan 手势是向右滑动的时候才做响应。

``` objc
/**
 *  指定 pan 的方向
 */
@property (nonatomic, assign) PanDirection direction;
/**
 *  该标志用于判断是否当前正在滑动中
 */
@property (nonatomic, assign) BOOL dragging;

```

注意：`#import <UIKit/UIGestureRecognizerSubclass.h>`
``` objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    [super touchesBegan:touches withEvent:event];
    self.dragging = NO;
}

- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
{
    [super touchesMoved:touches withEvent:event];
    if (self.state == UIGestureRecognizerStateFailed)
    {
        return;
    }

    CGPoint velocity = [self velocityInView:self.view];

    // 只检测一次
    if (!self.dragging)
    {
        NSDictionary *velocities = @{
                                     @(PanDirectionRight) : @(velocity.x),
                                     @(PanDirectionDown) : @(velocity.y),
                                     @(PanDirectionLeft) : @(-velocity.x),
                                     @(PanDirectionUp) : @(-velocity.y)
                                     };
        NSArray *keysSorted = [velocities keysSortedByValueUsingSelector:@selector(compare:)];

        // 如果 pan 的方向和指定的方向不同，那么状态为 fail
        if ([[keysSorted lastObject] integerValue] != self.direction)
        {
            self.state = UIGestureRecognizerStateFailed;
        }

        self.dragging = YES;
    }
}
```

3、定制切换动画效果
为 UIView 添加了一个扩展方法：使得 pop 出的视图在左侧边缘添加一个阴影，且随着移动渐变。
``` objc
- (void)addLeftSideShadowWithFading
{
    // 在视图的左侧边缘添加一个阴影效果
    CGFloat shadowWidth = 4.0f;
    CGFloat shadowVerticalPadding = -20.0f;
    CGFloat shadowHeight = CGRectGetHeight(self.frame) - 2 * shadowVerticalPadding;
    CGRect shadowRect = CGRectMake(-shadowWidth, shadowVerticalPadding, shadowWidth, shadowHeight);
    UIBezierPath *shadowPath = [UIBezierPath bezierPathWithRect:shadowRect];
    self.layer.shadowPath = [shadowPath CGPath];
    self.layer.shadowOpacity = 0.1f;

    // 在动画执行过程中，阴影透明度降低
    CGFloat toValue = 0.0f;
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"shadowOpacity"];
    animation.fromValue = @(self.layer.shadowOpacity);
    animation.toValue = @(toValue);
    [self.layer addAnimation:animation forKey:nil];
    self.layer.shadowOpacity = toValue;
}
```
接着就是实现 `UIViewControllerAnimatedTransitioning` 协议中的方法了
``` objc
- (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext
{
    return [transitionContext isInteractive] ? 0.4f : 0.5f;
}

// 注意： 这个动画是应用于 pop view controller 的时候，所以 fromVC 是需要 pop 的 VC (位于上层)，而 toVC 是需要展现的 VC (位于下层)。
- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext
{
    UIViewController *toViewController = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
    UIViewController *fromViewController = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    // toVC 置于 fromVC 的下面
    [[transitionContext containerView] insertSubview:toViewController.view belowSubview:fromViewController.view];

    // 增加视觉效果：toVC 从左往右移动。那么先做一个左移动
    CGFloat toViewControllerXTranslation = - CGRectGetWidth([transitionContext containerView].bounds) * 0.3f;
    toViewController.view.transform = CGAffineTransformMakeTranslation(toViewControllerXTranslation, 0);


    // 为 fromVC 添加一个左侧阴影
    [fromViewController.view addLeftSideShadowWithFading];
    BOOL previousClipsToBounds = fromViewController.view.clipsToBounds;
    fromViewController.view.clipsToBounds = NO;

    // 为 toVC 添加一个视图 -- 用于视觉效果（蒙层）
    UIView *dimmingView = [[UIView alloc] initWithFrame:toViewController.view.bounds];
    dimmingView.backgroundColor = [UIColor colorWithWhite:0.0f alpha:0.1f];
    [toViewController.view addSubview:dimmingView];

    [UIView animateWithDuration:[self transitionDuration:transitionContext] delay:0 options:UIViewAnimationOptionCurveLinear animations:^{

        toViewController.view.transform = CGAffineTransformIdentity;
        fromViewController.view.transform = CGAffineTransformMakeTranslation(toViewController.view.frame.size.width, 0);
        dimmingView.alpha = 0.0f;

    } completion:^(BOOL finished) {
        [dimmingView removeFromSuperview];
        fromViewController.view.transform = CGAffineTransformIdentity;
        fromViewController.view.clipsToBounds = previousClipsToBounds;
        [transitionContext completeTransition:![transitionContext transitionWasCancelled]];

    }];

    self.toViewController = toViewController;
}

- (void)animationEnded:(BOOL)transitionCompleted
{
    // 当切换动画取消时，重置 toViewController 的 transform
    if (!transitionCompleted)
    {
        self.toViewController.view.transform = CGAffineTransformIdentity;
    }
}
```

3、添加 Pan 手势 和  `UINavigationControllerDelegate` 委托方法中的处理

我们创建一个类：
`@interface SwipeToPop : NSObject <UINavigationControllerDelegate>`

我们传入一个 NavigationController 对象，然后为其添加 Pan 手势

``` objc
- (instancetype)initWithNavigationController:(UINavigationController *)navigationController
{
    self = [super init];
    if (self)
    {
        self.naviController = navigationController;

        // pan 手势
        DirectionalPanGesture *panGesture = [[DirectionalPanGesture alloc] initWithTarget:self action:@selector(handlePanGesture:)];
        panGesture.maximumNumberOfTouches = 1;
        // 指定向右滑动才响应
        panGesture.direction = PanDirectionRight;
        self.panGesture = panGesture;
        [self.naviController.view addGestureRecognizer:panGesture];

        // pop 动画效果
        self.popAnimation = [[PopAnimation alloc] init];
    }
    return self;
}

- (void)handlePanGesture:(UIPanGestureRecognizer *)recognizer
{
    UIView *view = self.naviController.view;

    if (recognizer.state == UIGestureRecognizerStateBegan)
    {
        if (self.naviController.viewControllers.count > 1 && !self.duringAnimation)
        {
            self.interaction = [[UIPercentDrivenInteractiveTransition alloc] init];
            self.interaction.completionCurve = UIViewAnimationCurveEaseOut;

            [self.naviController popViewControllerAnimated:YES];
        }
    }
    else if (recognizer.state == UIGestureRecognizerStateChanged)
    {
        CGPoint translation = [recognizer translationInView:view];
        // 更新交互动作完成百分比
        CGFloat d = translation.x > 0 ? translation.x / CGRectGetWidth(view.bounds) : 0;
        [self.interaction updateInteractiveTransition:d];
    }
    else if (recognizer.state == UIGestureRecognizerStateCancelled)
    {
        [self.interaction cancelInteractiveTransition];
        self.interaction = nil;
    }
    else if (recognizer.state == UIGestureRecognizerStateEnded)
    {
        if ([recognizer velocityInView:view].x > 0)
        {
            [self.interaction finishInteractiveTransition];
        }
        else
        {
            [self.interaction cancelInteractiveTransition];
            self.duringAnimation = NO;
        }
        self.interaction = nil;
    }
}
```

然后，实现 `UINavigationControllerDelegate` 的委托方法
``` objc
#pragma mark - UINavigationControllerDelegate
- (id<UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController animationControllerForOperation:(UINavigationControllerOperation)operation fromViewController:(UIViewController *)fromVC toViewController:(UIViewController *)toVC
{
    // 当事进行 pop 的时候，才返回动画
    if (operation == UINavigationControllerOperationPop)
    {
        return self.popAnimation;
    }
    return nil;
}

- (id<UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController interactionControllerForAnimationController:(id<UIViewControllerAnimatedTransitioning>)animationController
{
    // 返回切换交互动作
    return self.interaction;
}

- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    if (animated)
    {
        self.duringAnimation = YES;
    }
}

- (void)navigationController:(UINavigationController *)navigationController didShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    self.duringAnimation = NO;

    if (navigationController.viewControllers.count <= 1)
    {
        self.panGesture.enabled = NO;
    }
    else
    {
        self.panGesture.enabled = YES;
    }
}
```

最后在 FirstViewController 中补充：
``` objc
// navigation delegate
self.naviDelegate = [[SwipeToPop alloc] initWithNavigationController:self.navigationController];
self.navigationController.delegate = self.naviDelegate;
```

## 参考链接：
[iOS7中的ViewController切换 ](iOS7中的ViewController切换)
[Github SloppySwiper](https://github.com/fastred/SloppySwiper)
