title: JS继承与原型链
date: 2017-02-17 21:13:32
tags: [继承, 原型链]
categories: JS
---


案例分析：
``` js
function Thing1() {
    this.foo = 'foo'
}
Thing1.prototype.foo = "bar";

function Thing2() {
  this.logFoo(); // log: bar
  Thing1.apply(this);
  this.logFoo(); // log: foo
}
Thing2.prototype = Object.create(Thing1.prototype);
Thing2.prototype.constructor = Thing2;
Thing2.prototype.logFoo = function() {
  console.log(this.foo);
}

var thing2 = new Thing2();
```
上面这个例子中，依次输出：'bar', 'foo'。其中涉及到 JavaScript 继承及原型链，本文介绍下其中涉及的概念和作用机制。

<!-- more -->

在 JavaScript 中，一切皆对象(Object)，但是 JS 中并没有类(class)；*[ECMAScript6 引入 class,但其仍然是基于原型的]。* JS 是基于原型(prototype-based)来实现面向对象的编程范式。

在 JavaScript 中，每一个对象都对应一个原型对象，并从原型对象继承属性和方法；而这个原型对象又有自己的原型，直到某个对象的原型为null为止（也就是不再有原型指向），指向这条链的最后一环。这种一级一级的链结构称之为原型链(prototype-chain)。

## 关于 `__proto__` 和 `prototype`
- `__proto__` 属性的值就是对象对应的原型对象，标识自己所继承的原型

> 遵循ECMAScript标准，`someObject.[[Prototype]]` 符号是用于指派 `someObject` 的原型。这个等同于 JavaScript 的 `__proto__`  属性。从 ECMAScript 6 开始, `[[Prototype]]` 可以用 `Object.getPrototypeOf()` 和 `Object.setPrototypeOf()` 访问器来访问。

```
// ES 规范定义对象字面量的原型就是 Object.prototype
var one = {x: 1};
// 调用 Object 构造函数，所以 two.__proto__ 就是 Object.prototype
var two = new Object({x: 1});
one.__proto__ === Object.prototype // true
Object.getPrototypeOf(one) === Object.prototype // true
two.__proto__ === Object.prototype // true
Object.getPrototypeOf(two) === Object.prototype // true
```


- `prototype` 属性并不是每个对象都有（`__proto__`属性是每个对象都有），**只有函数才有 `prototype` 属性**

当你创建函数时，JS会为这个函数自动添加 `prototype` 属性，值是空对象。而一旦你把这个函数当作构造函数（constructor）调用（即通过new关键字调用），那么JS就会帮你创建该构造函数的实例，实例继承构造函数 `prototype` 的所有属性和方法（实例通过设置自己的 `__proto__` 指向承构造函数的 `prototype` 来实现这种继承）

```
function classA() {}
var a = new classA();

a.__proto__ === classA.prototype // true
```

当执行 `var a = new classA();` 时，JavaScript 实际执行的是：
```
var a = new Object();
a.[[Prototype]] = classA.prototype;
classA.call(a);
```
然后当你在访问实例的属性时，如 `a.someProp`，JavaScript 首先检查它们是否直接存在于该对象中（即是否是该对象的自身属性），如果没有，它会在其 `__proto__` 中查找，即 `Object.getPrototypeOf(a).someProp`，如果仍旧没有，它就会继续查找 `Object.getPrototypeOf(Object.getPrototypeOf(a)).someProp`，一直查找下去，直到它找到这个属性，或者 `Object.getPrototypeOf()` 返回 `null` 。


## 基于原型链的继承
JavaScript 中通过原型链实现继承，其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。回顾一下构造函数(constructor)，原型(prototype)和实例(instance)的关系：
- 每个构造函数都有一个原型对象(`prototype`)，原型对象都包含一个指向构造函数的指针(`constructor`)
- 实例都包含一个指向原型对象的内部指针(`__proto__`)

如果让原型对象等于另一个类型的实例，那么此时的原型对象将包含一个指向另一个原型的指针，如此层层递进，就构成了所谓的原型链。

下面详细介绍两种基于原型链继承的写法：
### 方式一

```
function Person(name) {
    this.name = name;
}
Person.prototype.sayName = function() {
    console.log(this.name)
}

function Programmer(name, age) {
    Person.call(this, name); // 第二次调用 Person() 构造函数

    this.age = age;
}

Programmer.prototype = new Person(); // 第一次调用 Person() 构造函数
Programmer.prototype.constructor = Programmer;
Programmer.prototype.sayAge = function() {
    console.log(this.age)
}
```
首先看一下 `Person`， 在构造函数中，定义了一个属性 `name`，然后在 `Person` 的原型中增加了一个 `sayName` 方法。
![](http://cdn.objcer.com/6AB5B2C2-FA48-4488-BA42-857B30F7CC35.png)

调用 `Person` 的构造函数，`var person = new Person('Tom');`，创建一个实例，我们可以得到：
![](http://cdn.objcer.com/253EFEA4-F163-471A-87EC-78D7781F57C0.png)
![](http://cdn.objcer.com/D7622C01-59F4-4EFD-9B93-D28D531E4E5A.png)

接着看一下子类 `Programmer`，调用构造函数 `var programmer = new Programmer('Jack', 20);` 创建一个实例对象。注意到其中的注释，我们发现，调用了两次 `Person` 的构造函数。第一次调用 `Person` 构造函数时，`Programmer.prototype` 会得到一个 `name` 属性，它是 `Person` 的实例属性，只不过现在位于 `Programmer` 的原型中。当调用 `Programmer` 的构造函数时，又一次调用了 `Person` 的构造函数，这一次又在新对象上创建了 `name` 属性，只不过这个 `name` 属性屏蔽了原型中的同名属性。

这时就会发现，有两个 `name` 属性，一个在实例上，一个在 `Programmer` 原型中，这就是两次调用 `Person` 构造函数的结果。

![](http://cdn.objcer.com/CF18A411-AFF9-41F8-9AC9-47FD7DEE6465.png)

### 方式二
那么其实不必为了指定子类型的原型而调用超类的构造函数，我们需要的只是超类型原型的一个副本而已。对其中的一行代码改造一下
```
// 将 Programmer.prototype = new Person(); 这行代码改成如下：
Programmer.prototype = Object.create(Person.prototype); // ①
Programmer.prototype.constructor = Programmer; // ②
```
如上，第①步创建超类型原型的一个副本，赋值给子类型的原型；第②步为子类型原型重新设置 constructor 属性，从而弥补因重写原型而失去的默认 constructor 属性。
这种继承方式的高效体现在其只调用了一次 `Person` 构造函数，并且因此避免了在 `Programmer.prototype` 上创建不必要的，多余的属性。所以这种方式也是目前继承最理想，最常用的写法。

### 其他
在原型链上查找属性比较耗时，对性能有副作用，这在性能要求苛刻的情况下很重要。另外，试图访问不存在的属性时会遍历整个原型链。遍历对象的属性时，原型链上的每个可枚举属性都会被枚举出来。
检测对象的属性是定义在自身上还是在原型链上，有必要使用 `hasOwnProperty` 方法，所有继承自 Object.proptotype 的对象都包含这个方法。`hasOwnProperty` 是 JavaScript 中唯一一个只涉及对象自身属性而不会遍历原型链的方法

所以，如上 `programmer` 实例：
```
programmer.hasOwnProperty('sayAge') // false
programmer.hasOwnProperty('sayName') // false
programmer.hasOwnProperty('name') // true
programmer.hasOwnProperty('age') // true
```

> 仅仅通过判断值是否为 undefined 还不足以检测一个属性是否存在，一个属性可能存在而其值恰好为 undefined

## 剖析开头案例

`Thing1` 原型中含有一个属性 `foo`，值为 `bar`；`Thing2` 继承 `Thing1`，也即是 `Thing2` 的原型对象指向 `Thing1` 的原型，那么也就继承了 `foo` 属性；接着 `Thing2` 原型增加了一个 `logFoo` 方法。

在 `Thing2` 构造函数中，第一次调用 `this.logFoo();` 时，查找 `foo` 属性，在自身对象中没有找到，接着往上找，`Object.getPrototypeOf(thing2).foo`，找到了，值为 `bar`。

然后，执行 `Thing1.apply(this);`，相当于调用了父类 `Thing1` 的构造函数，此时为 `thing2` 实例增加了一个 `foo` 属性，值为 `foo`。

最后，再一次调用 `this.logFoo();` ，查找 `foo` 属性，在自身对象中找到，值为 `foo`。

## 参考链接
[JavaScript Prototypal Inheritance](https://coryrylan.com/blog/javascript-prototypal-inheritance)
[MDN·继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)









