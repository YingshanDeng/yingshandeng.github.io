title: 'Debouncing and Throttling '
date: 2016-11-06 17:39:45
tags: [Debounce, Throttle]
categories: JS
---


Debounce 和 Throttle 是用于解决请求和响应速度不匹配问题的两中策略。例如对于窗口大小调整（resize 事件），页面滚动（scroll 事件）等触发回调频率高的问题，如果回调函数处理复制，那么势必会影响响应速度，往往就会导致卡顿，不流畅的问题。在工具类库 [underscore](https://github.com/jashkenas/underscore) 和 [lodash](https://github.com/lodash/lodash) 都提供了这两个函数。

<!-- more -->

## 问题引入
在 Twitter 页面就曾经遇到一个滚动事件频繁调用，导致页面不流畅的问题。具体描述和解决方式参考 [这篇文章](http://ejohn.org/blog/learning-from-twitter/#postcomment)。 其中提到了两个优化步骤：

① 不要将事件逻辑和滚动事件绑定在一起
② 将 DOM 节点查询结果进行缓存，方便重用

```
var outerPane = $details.find(“.details-pane-outer”),
didScroll = false;

$(window).scroll(function() {
  didScroll = true;
});

setInterval(function() {
  if ( didScroll ) {
    didScroll = false;
    // Check your page position and then
    // Load in more results
  }
}, 250);
```

## Debounce
> The Debounce technique allow us to "group" multiple sequential calls in a single one.

### 理解 Debounce 策略

用电梯运行来理解 Debounce。想象你在电梯中，电梯超时时间快结束，电梯门快要关闭的时候，突然又有一个人按了电梯要进入电梯，那么电梯就又打开了，超时时间重新倒计时，直到超时时间完全结束时，电梯才进行运送。可以结合如下图示进行理解：
![](http://cdn.objcer.com/debounce-trailing.png)

如上图，你可能会发现，当事件触发没有那么频繁，超过一定间隔后才执行一次回调函数；那么能否在事件触发时就立即执行回调函数，而在没有超过一定间隔时，不再进行调用回到函数。当然可以啦 👊 `Debounce` 函数中我们可以指定一个参数（有些称之为 `immediate` 或者 `leading`）默认 `Debounce` 函数是尾调用的。
![](http://cdn.objcer.com/debounce-leading.png)

```
// underscore.js
_.debounce = function(func, wait, immediate)
```

### Debounce 应用例子
1、窗口大小调整 `resize` 事件
```
$(window).on('resize', _.debounce(function() {
    // resize handler
 }, 400));
```

2、在搜索框中监听输入事件，在结束输入后，自动发起 ajax 请求

### 使用 Debounce 的陷阱与技巧
1、陷阱：有可能会将 `_.debounce` 多次调用
```
// WRONG
$(window).on('scroll', function() {
   _.debounce(doSomething, 300);
});

// RIGHT
$(window).on('scroll', _.debounce(doSomething, 200));
```

2、技巧：`debounce` 的私有方法 `cancel`
```
var debounced_version = _.debounce(doSomething, 200);
$(window).on('scroll', debounced_version);

// If you need it
debounced_version.cancel();
```

## Throttle
> By using _.throttle, we don't allow to our function to execute more than once every X milliseconds

### 理解 Throttle 策略
同样，用电梯运行来理解 Throttle。电梯中进入第一个人后，超时时间后，电梯准时运行，不等待。确保在规定的时间内（超时时间）执行一次。

### Throttle 应用例子
无限滚动（或者滚动事件监听）。在一个可以无限滚动的列表中，你需要检查是否快滚动到页面的底部，若是，需要执行相关的数据加载逻辑（ajax 请求），然后添加内容到列表，确保可以继续滚动。
在这个应用情景中，使用 Debounce 是不太恰当的，因为 Debounce 需要在滚动停止后的一个固定间隔后才执行相关的回调处理逻辑，而这显然不符合当前的操作体验；而 Throttle 在固定的时间间隔中都执行一次相关的逻辑，符合要求。

```
$(document).on('scroll', _.throttle(function(){
  check_if_needs_more_content();
}, 300));
```

### 使用 Throttle 的技巧
```
// underscore.js
_.throttle = function(func, wait, options)
```
默认情况下，throttle 将在你调用的第一时间尽快执行这个 function，并且，如果你在 wait 周期内调用任意次数的函数，都将尽快的被覆盖。如果你想禁用第一次首先执行的话，`options` 传递{leading: false}，还有如果你想禁用最后一次执行的话，传递{trailing: false}。

## requestAnimationFrame (rAF)

`requestAnimationFrame` 是另一种控制函数调用频率的方式，其可以理解成 `_.throttle(dosomething, 16)` 在重新计算节点位置，重新渲染，执行动画等时使用会有较好的效果。

## 总结

`debounce`, `throttle` 和 `requestAnimationFrame` 都可以优化事件回调的调用，各有特点，结合实际情况进行使用。

- **debounce**: Grouping a sudden burst of events (like keystrokes) into a single one.
- **throttle**: Guaranteeing a constant flow of executions every X milliseconds. Like checking every 200ms your scroll position to trigger a CSS animation.
- **requestAnimationFrame**: a throttle alternative. When your function recalculates and renders elements on screen and you want to guarantee smooth changes or animations. Note: no IE9 support.

--------
注: 以上文章主要是 [Debouncing and Throttling Explained Through Examples](https://css-tricks.com/debouncing-throttling-explained-examples/) 的理解后关键内容的提取和总结。
TODO: 完善例子
