title: Implement Throttling and Debouncing in JavaScript
tags:
---

Debouncing 和 Throttling 提供了控制函数执行频率的能力，使得我们在鼠标移动，窗口大小调整，页面滚动等事件回调频繁调用的场景下，可以通过这两个函数进行优化。在文章 [Debouncing and Throttling](https://objcer.com/2016/11/06/Debouncing-and-Throttling/) 中介绍了 Debouncing 和 Throttling 的用法，而本文将介绍 Debouncing 和 Throttling 源码如何实现 🤓

<!-- more -->

## Debounce
```
const debounce = (func, delay) => {
  let inDebounce
  return function() {
    const context = this
    const args = arguments
    clearTimeout(inDebounce)
    inDebounce = setTimeout(() => func.apply(context, args), delay)
  }
}
```
debounce 将延迟函数的执行(真正的执行)在函数最后一次调用时刻的 delay 毫秒之后。如下例子，连续点击 button (且间隔小于 1000 毫秒)，那么回调函数将会在最后一次点击延时 1000 毫秒执行。
```
var button = document.getElementById('btn')
button.addEventListener('click', debounce((evt) => {
  console.log(evt)
}, 1000))
```

debouce 在 underscore 或者 lodash 第三方库中都有实现：
```
// 传参 immediate 为 true，debounce会在 wait 时间间隔的开始调用这个函数
_.debounce(function, wait, [immediate])
```

下面给出 underscore 中的实现：
```js
/**
 * Returns a function, that, as long as it continues to be invoked, will not
 * be triggered. The function will be called after it stops being called for
 * N milliseconds. If `immediate` is passed, trigger the function on the
 * leading edge, instead of the trailing. The function also has a property 'clear'
 * that is a function which will clear the timer to prevent previously scheduled executions.
 *
 * @source underscore.js
 * @see http://unscriptable.com/2009/03/20/debouncing-javascript-methods/
 * @param {Function} function to wrap
 * @param {Number} timeout in ms (`100`)
 * @param {Boolean} whether to execute at the beginning (`false`)
 * @api public
 */

function debounce(func, wait, immediate) {
  var timeout, args, context, timestamp, result;
  if (null == wait) wait = 100;

  function later() {
    var last = Date.now() - timestamp;

    if (last < wait && last >= 0) { // 确保不会提前调用
      timeout = setTimeout(later, wait - last);
    } else {
      timeout = null;
      if (!immediate) { // trigger the function on the trailing edge
        result = func.apply(context, args);
        context = args = null;
      }
    }
  };

  var debounced = function(){
    context = this;
    args = arguments;
    timestamp = Date.now();
    var callNow = immediate && !timeout;
    if (!timeout) timeout = setTimeout(later, wait);
    if (callNow) { // trigger the function on the  leading edge
      result = func.apply(context, args);
      context = args = null;
    }

    return result;
  };

  // 辅助方法 clear/flush
  debounced.clear = function() {
    if (timeout) {
      clearTimeout(timeout);
      timeout = null;
    }
  };
  debounced.flush = function() {
    if (timeout) {
      result = func.apply(context, args);
      context = args = null;

      clearTimeout(timeout);
      timeout = null;
    }
  };

  return debounced;
};
```

## Throttle
```
const throttle = (func, delay) => {
  let inThrottle
  return function() {
    const args = arguments
    const context = this
    if (!inThrottle) {
      func.apply(context, args)
      inThrottle = true
      setTimeout(() => inThrottle = false, delay)
    }
  }
}
```
throttle 像一个节流阀一样，当重复调用函数的时候，至少每隔 delay 毫秒调用一次该函数，这对于一些触发频率较高的事件有帮助。如下例子，连续点击 button，那么回调函数将会每间隔 1000 毫秒执行一次(注意和 debounce 的区别)。
```
var button = document.getElementById('btn')
button.addEventListener('click', throttle((evt) => {
  console.log(evt)
}, 1000))
```

注意到上面的 throttle 实现中，连续点击 button，第一次调用回调函数之后间隔 1000 毫秒才会调用第二次。那么可能会出现这样的问题，连续点击持续的时间小于 1000 毫秒，那么这段时间内的点击事件都被忽略了，如下图所示，紫色箭头表示调用了回调函数，注意到阴影覆盖的两处调用被忽略了。
![](http://7vikhl.com1.z0.glb.clouddn.com/2685DF6F-96CA-43A8-A037-7BF4F163E2AB.png)

例如对于 `drag` 或者 `resize` 作用的鼠标移动事件，如果忽略了最后部分的调用，那么可能会导致最终结果的不正确，所以我们需要增加处理，对最后部分的调用在间隔结束后，也要执行回调函数。如下：
```
const throttle = (func, limit) => {
  let inThrottle
  let lastFunc
  let lastRan
  return function() {
    const context = this
    const args = arguments
    if (!inThrottle) {
      func.apply(context, args)
      lastRan = Date.now()
      inThrottle = true
    } else {
      clearTimeout(lastFunc)
      lastFunc = setTimeout(function() {
        if ((Date.now() - lastRan) >= limit) {
          func.apply(context, args)
          lastRan = Date.now()
        }
      }, limit - (Date.now() - lastRan))
    }
  }
}
```

throttle 在 underscore 或者 lodash 第三方库中都有实现：
```
_.throttle(function, wait, [options])
```
默认情况下，throttle将在你调用的第一时间尽快执行这个 function，并且，如果你在 wait 周期内调用任意次数的函数，都将尽快的被覆盖。如果你想禁用第一次首先执行的话，传递 `{leading: false}`，还有如果你想禁用最后一次执行的话，传递 `{trailing: false}`。

下面给出 underscore 中的实现：
```
// Returns a function, that, when invoked, will only be triggered at most once
// during a given window of time. Normally, the throttled function will run
// as much as it can, without ever going more than once per `wait` duration;
// but if you'd like to disable the execution on the leading edge, pass
// `{leading: false}`. To disable execution on the trailing edge, ditto.
function throttle(func, wait, options) {
  var context, args, result;
  var timeout = null;
  var previous = 0;
  if (!options) options = {};
  var later = function() {
    previous = options.leading === false ? 0 : Date.now();
    timeout = null;
    result = func.apply(context, args);
    if (!timeout) context = args = null;
  };
  return function() {
    var now = Date.now();
    if (!previous && options.leading === false) previous = now;
    var remaining = wait - (now - previous);
    context = this;
    args = arguments;
    if (remaining <= 0 || remaining > wait) {
      if (timeout) {
        clearTimeout(timeout);
        timeout = null;
      }
      previous = now;
      result = func.apply(context, args);
      if (!timeout) context = args = null;
    } else if (!timeout && options.trailing !== false) {
      timeout = setTimeout(later, remaining);
    }
    return result;
  };
};
```

## 参考链接
[Throttling and Debouncing in JavaScript](https://medium.com/@_jh3y/throttling-and-debouncing-in-javascript-b01cad5c8edf)
