title: 探究数据绑定（2）ES6 Proxy
date: 2017-10-31 15:25:45
tags: [数据绑定, Proxy]
categories: JS
---


![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/data-binding-with-es6-proxy.png)
在上一篇文章：[探究数据绑定（1）脏检查和存取器方法](https://objcer.com/2017/10/27/Data-binding-introduction/) 中，探究了 ES5 中实现数据绑定的两种方式；而在本文中，将会探究使用 ES6 Proxy 实现数据绑定。

<!-- more -->

## 知识储备
Proxy 方式实现数据绑定中涉及到 Proxy、Reflect、Set、Map 和 WeakMap，这些都是 ES6 的新特性。

### Proxy

Proxy 对象代理，在目标对象之前架设一层拦截，外部对目标对象的操作，都会通过这层拦截，我们可以定制拦截行为，每一个被代理的拦截行为都对应一个处理函数。
```
let p = new Proxy(target, handler);
```

```
var handler = {
  get: (target, name, recevier) => {
    return 'proxy'
  }
}
var p = new Proxy({}, handler)
p.a = 1

console.log(p.a, p.c) // -> proxy proxy
```

Proxy 构造函数接收两个参数：
- 第一个参数是要代理的目标对象
- 第二个参数是配置对象，每一个被代理的操作都对应一个处理函数

在这个例子中，目标对象是一个空对象，配置对象中有一个 `get` 函数，用来拦截外部对目标对象属性的访问，可以看到，`get` 函数始终返回 `proxy`。

Proxy 支持拦截的操作一共有13种：
- get(target, propKey, receiver)
- set(target, propKey, value, receiver)
- has(target, propKey)
- deleteProperty(target, propKey)
- ownKeys(target)
- getOwnPropertyDescriptor(target, propKey)
- defineProperty(target, propKey, propDesc)
- preventExtensions(target)
- getPrototypeOf(target)
- isExtensible(target)
- setPrototypeOf(target, proto)
- apply(target, object, args)
- construct(target, args)

> 👉 更详细介绍参考：
[MDN·Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
[Proxy](http://es6.ruanyifeng.com/#docs/proxy)

### Reflect
Reflect 对象同 Proxy 对象一样，也是 ES6 为了操作对象而提供的新特性。

Reflect 对象的方法与 Proxy 对象的方法一一对应，只要是 Proxy 对象的方法，就能在 Reflect 对象上找到对应的静态方法（Reflect 对象没有构造函数，不能使用 new 创建实例）。这就让 Proxy 对象可以方便地调用对应的 Reflect 方法，完成默认行为，作为修改行为的基础。也就是说，不管 Proxy 怎么修改默认行为，你总可以在 Reflect 上获取默认行为。
```
var handler = {
  get: (target, name, recevier) => {
    console.log('get: ', name)
    Reflect.get(target, name)
  },
  set: (target, name, value, recevier) => {
    console.log('set: ', name)
    Reflect.get(target, name, value)
  }
}
var p = new Proxy({}, handler)
p.a = 1

console.log(p.a, p.c)
```

代码执行结果，输出：
```
set:  a
get:  a
get:  c
```

上面代码中，Proxy 拦截目标对象的 `get` 和 `set`方法，在其中定制拦截行为，最后采用 `Reflect.get` 和 `Reflect.set` 分别完成目标对象默认的属性获取和设置行为。

> 👉 更详细介绍参考：
[MDN·Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
[Reflect](http://es6.ruanyifeng.com/#docs/reflect)

### Set/WeakSet 和 Map/WeakMap
**Set**
  - 类似 Array 数组
  - Set 允许你存储任何类型的唯一值，无论是原始值或者是对象引用
  - Set 成员的值都是唯一的，没有重复值

**WeakSet**
  - 类似 Set，也是不重复元素的集合
  - WeakSet 对象中只能存放对象值, 不能存放原始值, 而 Set 对象都可以
  - WeakSet 对象中存储的**对象值都是被弱引用的**, 如果没有其他的变量或属性引用这个对象值, 则这个对象值会被当成垃圾回收掉. 正因为这样, WeakSet 对象是无法被枚举的, 没有办法拿到它包含的所有元素

**Map**
  - 类似 Object 对象，保存键值对
  - Map 任何值(对象或者原始值) 都可以作为一个键(key)或一个值(value)，而 Object 对象的 key 键值只能是字符串

**WeakMap**
  - 类似 Map，也是一组键值对的集合
  - WeakMap 对象中的**键是弱引用的**。键必须是对象，值可以是任意值
  - 由于这样的弱引用，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用，原理同 WeakSet
  - WeakMap 的 key 是非枚举的

## Proxy 实现数据绑定
先上完整代码 👉
```
// 监听对象集合
var observers = new WeakMap()
// 待执行监听函数集合，Set 可以避免重复
var queuedObservers = new Set()
// 当前监听函数
var currentObserver

function observable(obj) {
  observers.set(obj, new Map())
  return new Proxy(obj, { get, set })
}

function get(target, key, receiver) {
  // get 方法默认行为
  const result = Reflect.get(target, key, receiver)
  // 当前监听函数中，监听使用了该属性，
  // 那么把该 监听函数 存放到该属性对应的 对象属性监听函数集合 Set
  if (currentObserver) {
    registerObserver(target, key, currentObserver)
  }
  return result
}
function registerObserver(target, key, observer) {
  let observersForKey = observers.get(target).get(key)
  // 为每一个对象属性都创建一个 Set 集合，存放监听了该属性的监听函数
  if (!observersForKey) {
    observersForKey = new Set()
    observers.get(target).set(key, observersForKey)
  }
  observersForKey.add(observer)
}

function set(target, key, value, receiver) {
  const observersForKey = observers.get(target).get(key)
  // 修改对象属性，即对象属性值发生变更时，
  // 判断 对象属性监听函数集合 Set 是否存在，将其中的所有监听函数都添加到 待执行监听函数集合
  if (observersForKey) {
    observersForKey.forEach(queueObserver)
  }
  // set 方法默认行为
  return Reflect.set(target, key, value, receiver)
}

function observe(fn) {
  queueObserver(fn)
}

// 将监听函数添加到 待执行监听函数集合 Set 中
// 如果 待执行监听函数集合 Set 为空，那么在添加后立即执行
function queueObserver(observer) {
  if (queuedObservers.size === 0) {
    // 异步执行
    Promise.resolve().then(runObservers)
  }
  queuedObservers.add(observer)
}

// 执行 待执行监听函数集合 Set 中的监听函数
// 执行完毕后，进行清理工作
function runObservers() {
  try {
    queuedObservers.forEach((observer) => {
      currentObserver = observer
      observer()
    })
  } finally {
    currentObserver = undefined
    queuedObservers.clear()
  }
}
```

对外暴露的 `observable(obj)` 和 `observe(fn)` 方法二者分别用于创建 observable 监听对象和 observer 监听回调函数。当 observable 监听对象发生属性变化时，observer 函数将自动执行。

测试用例：
```
var obj = {name: 'John', age: 20}
// observable object
var person = observable(obj)

function print () {
  console.log(`监听属性发生变化：${person.name}, ${person.age}`)
}
// observer function
observe(print)
```

### 分析接口方法
关于 `observable(obj)` 和 `observe(fn)`：
**`observable(obj)` 方法中，通过 ES6 Proxy 为目标对象 `obj` 创建代理，拦截 `get` 和 `set` 操作**
- 当前监听函数：`currentObserver`
- 待执行监听函数集合 Set：`var queuedObservers = new Set()`
- 监听对象集合 WeakMap：`var observers = new WeakMap()` 键值为监听对象
- 对象属性监听函数集合 Set：监听了对象属性的监听函数，都保存到对象属性监听函数集合 Set 中，方便在对象属性发生变更时，执行监听函数
- 拦截方法 `get`：使用 `obj.property` 获取对象属性，即会被拦截方法 `get` 拦截
  👉 查看 `get` 中的注释
- 拦截方法 `set`：使用 `obj.property = value` 设置对象属性，即会被拦截方法 `set` 拦截
  👉 查看 `set` 中的注释

**`observe(fn)` 方法中，添加对象属性监听函数**
  监听函数中使用 `obj.property` 获取对象属性，即表明监听函数监听了该属性，那么就会触发拦截方法 `get` 中对监听属性的逻辑处理，为其创建对象属性监听函数集合 Set，并将当前的监听函数添加进其中


### 分析测试用例
下面通过流程图讲解一下测试用例的执行过程
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/es6-proxy-observables-code.png)

- 通过 `observable` 方法创建代理对象 `person`
- `observe` 方法设置监听函数，此时待执行监听函数集合 Set 为空，监听函数添加到 Set 中后执行待执行监听函数集合 Set 中的监听函数
- 在 `runObservers` 方法中当前监听函数 `currentObserver` 被设为 `print`
- `print` 开始执行
- 在 `print` 内部检索到 `person.name`
- 在 `person` 上触发拦截方法 `get`
- `observers.get(person).get('name')` 检索到 `(person, name)` 组合的对象属性监听函数集 Set
- 当前监听函数 `print` 被添加到对象属性监听函数集 Set 中
- 对于 `person.age`，同理，执行前面在 `print` 内部检索到 `person.name` 的流程
- `${person.name}, ${person.age}` 打印出来；
- `print` 函数执行结束；
- 当前监听函数 `currentObserver` 变为 `undefined`

当调用 `person.age = 22` 修改对象属性时：
- `person` 上触发拦截方法 `set`
- `observers.get(person).get('age')` 检索到 `(person, age)` 组合的对象属性监听函数集 Set
- 对象属性监听函数集 Set 中的监听函数（包括 `print`）入待执行监听函数集合，准备执行
- 再次执行 `print`

## 高级主题

### 动态 observable tree
```
var obj = {
  name: 'John',
  age: 20,
  teacher: {
    name: 'Tom',
    age: 30
}}
// observable object
var person = observable(obj)

function print () {
  console.log(`监听属性发生变化：${person.teacher.name}, ${person.teacher.age}`)
}
// observer function
observe(print)

setTimeout(() => {person.teacher.name = 'Jack'})
```
到目前为止，单层对象的数据绑定监听是正常工作的。但是在这个例子中，我们监听的对象值又是对象，这个时候监听就失效了，我们需要将：
```
observable({data: {name: 'John'}})
```

替换成
```
observable({data: observable({name: 'John'})})
```

这样就能正常运行了 😋

显然，这样使用不方便，可以做拦截方法 `get` 中修改一下，在返回值是对象时，对返回值对象也调用 `observable(obj)` 为其创建监听对象。
```
function get(target, key, receiver) {
  const result = Reflect.get(target, key, receiver)
  if (currentObserver) {
    registerObserver(target, key, currentObserver)
    if (typeof result === 'object') {
      const observableResult = observable(result)
      Reflect.set(target, key, observableResult, receiver)
      return observableResult
    }
  }
  return result
}
```

### 继承
对于 Proxy 拦截操作也可以在原型链中被继承，例如：
```
let proto = new Proxy({}, {
  get(target, propertyKey, receiver) {
    console.log('GET ' + propertyKey);
    return Reflect.get(target, propertyKey, receiver);
  }
});

let obj = Object.create(proto);
obj.foo // "GET foo"
```

上面代码中，拦截操作 `get` 定义在原型对象上面，所以如果读取 `obj` 对象属性时，拦截会生效。

同理，通过 Proxy 实现的数据绑定也能与原型继承搭配工作，例如：
```
const parent = observable({greeting: 'Hello'})
const child = observable({subject: 'World!'})
Object.setPrototypeOf(child, parent)

function print () {
  console.log(`${child.greeting} ${child.subject}`)
}

// 控制台打印出 'Hello World!'
observe(print)

// 控制台打印出 'Hello There!'
setTimeout(() => child.subject = 'There!')

// 控制台打印出 'Hey There!'
setTimeout(() => parent.greeting = 'Hey', 100)

// 控制台打印出 'Look There!'
setTimeout(() => child.greeting = 'Look', 200)
```

### 源码
本文中通过简单的代码展示了 Proxy 实现数据绑定，更加完整的实现，参考：[nx-js/observer-util](https://github.com/nx-js/observer-util)

## 参考链接
[Writing a JavaScript Framework - Data Binding with ES6 Proxies](https://blog.risingstack.com/writing-a-javascript-framework-data-binding-es6-proxy/)
[使用 ES6 Proxy 实现数据绑定](http://www.zcfy.cc/article/writing-a-javascript-framework-data-binding-with-es6-proxies-risingstack-1655.html)
