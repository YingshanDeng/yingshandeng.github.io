title: 探究数据绑定（1）脏检查和存取器方法
date: 2017-10-27 10:30:21
tags: [数据绑定, 脏检查, 存取器方法]
categories: JS
---

> **Data binding is a general technique that binds data sources from the provider and consumer together and synchronizes them.**

在前端组件化框架中，基本都有**数据绑定**这一特性，数据绑定是将生产者的数据源绑定到消费者，并负责数据同步。本文将探究数据绑定的不同实现方式。🤓

目前主要有以下几种实现方案：
- **脏检查（dirty check）**
- **存取器方法（accessor method）**: `Object.defineProperty` ES5
- `Proxy`: ES6 支持较差【Proxies are one of the few non polyfillable additions.（babel不支持转换）】
- `Object.observe`: ES7，已经移出草案，不支持
- `object.watch`: 目前只有基于gecko的浏览器如火狐支持,官方建议仅供调试用

其中前两种方式是主流，angular、Polymer 使用的就是脏检查，Vue 使用的就是存取器方法。

<!-- more -->

## 脏检查（dirty check）
通过一个例子来阐述脏检查的原理：
```
const provider = {
  message: 'Hello World'
}
const consumer = document.createElement('p')

observe(provider, 'message', message => {
  consumer.innerHTML = message
})
```

定义一个 `provider` 对象作为生产者，DOM 节点 `consumer` 作为消费者，通过 `observe` 监听，生产者数据的变更会反映到消费者上，这样也就简单的实现了数据绑定，即生产者数据绑定到了消费者上。
```
function observe (provider, prop, handler) {
  provider._handlers[prop] = handler
}
```

而其中监听函数 `observe` 只是将对某一个属性的监听回调 `handler` 保存起来，方便在该属性值发生变化时，执行该回调。

接下来就是监听变化过程了，脏检查之所以称之为*脏*，是因为其不是直接监听属性是否发生了变更，而是通过一个定时器轮询检查，不断的遍历检查对象的新值和旧值，判断是否发生了变化，若是，则调用会回调函数 `handler` 将生产者的数据同步到消费者上。
```
function digest () {
  providers.forEach(digestProvider)
}

function digestProvider (provider) {
  for (let prop in provider._handlers) {
    if (provider._prevValues[prop] !== provider[prop]) {
      provider._prevValues[prop] = provider[prop]
      handler(provider[prop])
    }
  }
}
```

**所以对于 `digest` 方法就需要循环进行调用**，譬如使用 `setInterval` 或者 `requestAnimationFrame` 方法，显然这个过程会对性能产生影响，当监听的对象越来越多时尤其明显。

在 [Polymer](https://www.polymer-project.org/) 1.0 版本中的数据绑定也是使用脏检查的解决方案，在其 `observe-js-behavior.html` 文件中我们就发现了通过定时器轮询脏检查的代码：
```
// Poll observe-js all the time
setInterval(Platform.performMicrotaskCheckpoint, 125);
```

而 angular 就对此也进行了优化，其并不使用定时轮询脏检查，而是对常用的 DOM 事件， XHR 事件等进行封装，在里面触发进入脏检查。

脏检查应该是实现对象监听比较成熟和完整的解决方案。可以参考以下两个项目:
- [Polymer/observe-js](https://github.com/Polymer/observe-js)
- [MaxArt2501/object-observe](https://github.com/MaxArt2501/object-observe)

而对于前者，也是 [Polymer](https://www.polymer-project.org/) 框架中使用的监听方案，

## 存取器方法（accessor method）
这种方式是通过重写对象（`Object`类型）属性的 `set` 和 `get` 方法，在 `setter` 方法中执行相应的监听回调（callback）来实现的。

对对象的监听需要考虑 `Object` 类型和 `Array` 类型这两种类型
```
// 情况一：obj1 只包含 `Object` 类型
var obj1 = {
  a: 1,
  b: 2,
  c: {
   d: 3,
   e: 4
  }
}
// 情况二：obj2 包含 `Object` 类型和 `Array` 类型
var obj2 = {
  a: 1,
  b: 2,
  c: [3, 4]
}
```
### Step 1
首先我们考虑情况一，对象中不包含 `Array` 类型
```
function Observer (obj, callback) {
  var observe = function (obj, path) {
    let type = Object.prototype.toString.call(obj);
    // Object
    if(type === '[object Object]') {
     observeObject(obj, path);
    }
  }
  var observeObject = function (obj, path) {
    // for...in 可以遍历对象的实例属性和原型属性，而 Object.keys 只遍历实例属性
    for (let prop in obj) {
      // 注意这里不要用 var
      let value = obj[prop],
        _path = path.slice(); // 主要对 path 数组进行一次拷贝
        _path.push(prop); // 记录路径
      Object.defineProperty(obj, prop, {
        set: function(newVle) {
          callback(_path, newVle, value);
          value = newVle;
        },
        get: function() {
          return value;
        }
      });
      // 递归
      observe(value, _path);
   }
  }
  observe(obj, []);
}
```

**📌注意**
- `for...in` 和 `Object.keys` 的区别，[参考链接](http://objcer.com/2017/02/22/js-enumerable/)
- `for...in` 循环中，变量定义不能使用 `var`，需要使用 `let` 或者用闭包，如下
```
...
for (let prop in obj) {
  (function() {
    var value = obj[prop],
        _path = path.slice();
    _path.push(prop); // 记录路径
    Object.defineProperty(obj, prop, {
      set: function(newVle) {
        callback(_path, newVle, value);
        value = newVle;
      },
      get: function() {
        return value;
      }
    });
    // 递归
    observe(value, _path);
  })()
}
...
```

测试代码：
```
var obj = {
  a: 1,
  b: 2,
  c: {
   d: 3,
   e: 4
  }
};
new Observer(obj, (path, newVle, oldVle) => {
  console.log(`path: ${path}, newValue: ${newVle}, oldValue: ${oldVle}`)
});

obj.a = 5; // log: path: a, newValue: 5, oldValue: 1
obj.c.d = 6; // log: path: c,d, newValue: 6, oldValue: 3
```
**💣 局限性**
通过 `Object.defineProperty` 实现对对象的监听，我们是在 `set` 方法中做文章，那么监听的情况也就只限于对象属性的修改（modify），如果对对象属性的增删（add/delete），那么就无能为力了。

### Step 2
接着我们考虑情况二，对象中包含 `Object` 类型和 `Array` 类型
```
function Observer (obj, callback) {
  var observe = function (obj, path) {
    let type = Object.prototype.toString.call(obj);
    if(type === '[object Object]' || type== '[object Array]') {
      observeObject(obj, path);
      if (type === '[object Array]') {
        observeArrayPreparation(obj, path);
      }
    }
  }
  var observeObject = function (obj, path) {
    // for...in 可以遍历对象的实例属性和原型属性，而 Object.keys 只遍历实例属性
    for (let prop in obj) {
      // 注意：不能用 var
      let value = obj[prop],
        _path = path.slice(); // 主要对 path 数组进行一次拷贝
        _path.push(prop); // 记录路径
      Object.defineProperty(obj, prop, {
        set: function(newVle) {
          callback(_path, newVle, value);
          value = newVle;
        },
        get: function() {
          return value;
        }
      });
       // 递归
       observe(value, _path);
    }
  }
  var observeArrayPreparation = function (arr, path) {
    var _props = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'];
    var _newProto = Object.create(Array.prototype);
    _props.forEach((prop) => {
      Object.defineProperty(_newProto, prop, {
        value: function () { // 注意：数组方法的参数个数不确定，所以此处暂不处理
          var _path = path.slice();
          _path.push(prop);
          // 注意：此处只返回 path
          callback(_path);
          Array.prototype[prop].apply(arr, arguments);
        }
      })
    });
    arr.__proto__ = _newProto;
  }
  observe(obj, []);
}
```

涉及到数组类型，解决的方法是，重新封装一下数组的操作方法 `['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse']`，在其中调用相应的监听回调

测试代码：
```
var obj = {
  a: 1,
  b: 2,
  c: [3, 4]
};
new Observer(obj, (path, newVle, oldVle) => {
  console.log(`path: ${path}, newValue: ${newVle}, oldValue: ${oldVle}`)
});

obj.c[1] = 33 // path: c,1, newValue: 33, oldValue: 4

obj.c.push(5); // path: c,push, newValue: undefined, oldValue: undefined
```
**📌注意**
- 由于数组不同操作方法的参数个数不同，所以在重新定义时需要对此进行判断处理，此处暂不处理，监听回调也只返回 path 路径
- 上述对数组的处理还存在很多问题，譬如测试代码中，新添加的 `5` 这个值并没有进行监听，如果此时对 `obj.c[2]` 进行修改，那么发现并没有回调监听函数；所以此处只提供一个思路。

## 参考链接
[Writing a JavaScript Framework - Introduction to Data Binding, beyond Dirty Checking](https://blog.risingstack.com/writing-a-javascript-framework-data-binding-dirty-checking/)
