title: 探究 JS 中的浅拷贝和深拷贝
date: 2017-02-27 00:09:14
tags: [深拷贝(deep clone), 浅拷贝(shallow clone)]
categories: JS
---


<!-- more -->

## 数据类型
JS 中数据类型可划分为
- Primitive Data Types(原始类型): **String, Number, Boolean, Null, Undefined**
- Composite(reference) Data Types(引用类型): **Object, Array**

> 1、其中 Undefined 类型和 Null 类型都只有一个值，分别是 undefined 和 null。
2、通过 `typeof` 运算符可以检查一个变量或者值的类型，例如变量是 String 类型，那么 `typeof` 得到的结果就是 `string`。但是有一个例外是 `typeof null` 的结果是 `object`。那为什么 `typeof` 运算符对于 null 值会返回 "Object"。这实际上是 JavaScript 最初实现中的一个错误，然后被 ECMAScript 沿用了。现在，null 被认为是对象的占位符，从而解释了这一矛盾，但从技术上来说，它仍然是原始值。

## 浅拷贝 shallow clone
在 JavaScript 中（ES6之前）是没有提供的对象拷贝的方法。拷贝一个对象最简单的方式是，创建一个空对象，然后遍历被拷贝对象的属性，拷贝到空对象上。
```
function naiveShallowCopy( original )  {
    // First create an empty object
    // that will receive copies of properties
    var clone = {} ;
    var key ;

    for ( key in original ) {
        // copy each property into the clone
        clone[ key ] = original[ key ] ;
    }
    return clone ;
}
```
然而，这段代码存在如下的问题：
- `naiveShallowCopy` 函数返回的拷贝对象 `clone` 和被拷贝对象 `original` 的原型对象可能不同，`clone` 对象只是一个普通的 `Object` 对象。
- 被拷贝对象 `original` 中的原型属性（来自原型对象中的属性）被拷贝到 `clone` 对象后，变成了实例属性（对象自身拥有的属性）
- 只有可枚举的属性被拷贝到 `clone` 对象中
- 对象属性的描述符(Properties' descriptor) 没有被拷贝，例如：被拷贝对象 `original` 中的一个只读属性在拷贝对象 `clone` 中变成了可读可写
- 最重要的是，如果对象中某个属性值是对象，那么经过上述代码拷贝后，拷贝对象 `clone` 和被拷贝对象 `original` 对象中对应的属性值指向同一个对象

而其中第五点，也就是 shallow clone 和 deep clone 的本质区别了。

> 关于属性描述符，属性的可枚举性，原型属性和实例属性，`for...in` 和 `Object.keys()` 的区别可以参考[这篇文章](http://objcer.com/2017/02/22/js-enumerable/#more)

对于 `for...in` 和 `Object.keys()` 遍历对象属性，二者是有区别的。`for...in ` 循环可以遍历对象中所有可枚举的对象属性，包括原型属性和实例属性(也就是继承的属性和对象自有属性)；而 `Object.keys` 方法只能遍历出实例属性(也就是对象本身自有的属性)。

如果只需要遍历拷贝对象的可枚举的自有属性（实例属性），我们可以使用 `Object.keys()`
```
function shallowCopyOfEnumerableOwnProperties( original )  {
    // First create an empty object
    // that will receive copies of properties
    var clone = {} ;
    var i , keys = Object.keys( original ) ;

    for ( i = 0 ; i < keys.length ; i ++ ) {
        // copy each property into the clone
        clone[ keys[ i ] ] = original[ keys[ i ] ] ;
    }
    return clone ;
}
```

如果需要将对象的可枚举和不可枚举属性都遍历出来进行拷贝，那么可以使用 `Object.getOwnPropertyNames(obj)`，这个方法返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性）组成的数组，但不会获取原型链上的属性。
```
function shallowCopyOfOwnProperties( original ) {
    // First create an empty object
    // that will receive copies of properties
    var clone = {} ;
    var i , keys = Object.getOwnPropertyNames( original ) ;

    for ( i = 0 ; i < keys.length ; i ++ ) {
        // copy each property into the clone
        clone[ keys[ i ] ] = original[ keys[ i ] ] ;
    }
    return clone ;
}
```

我们可以通过 `Object.getOwnPropertyDescriptor()` 和 `Object.defineProperty()` 方法解决拷贝过程中属性描述符丢失的问题；
此外，在创建拷贝对象 `clone` 时，使用 `Object.create()`， 并将 `Object.getPrototypeOf( original )` 作为其参数，这样就可以保证拷贝对象 `clone` 和被拷贝对象 `original` 的原型对象一致。
```
function shallowCopy( original ) {
    // First create an empty object with
    // same prototype of our original source
    var clone = Object.create( Object.getPrototypeOf( original ) ) ;
    var i , keys = Object.getOwnPropertyNames( original ) ;

    for ( i = 0 ; i < keys.length ; i ++ ){
        // copy each property into the clone
        Object.defineProperty( clone , keys[ i ] ,
            Object.getOwnPropertyDescriptor( original , keys[ i ] )
        ) ;
    }
    return clone ;
}
```
至此上述五个问题中，除了最后一个问题，都得到了解决，这样我们就近乎完美的手动实现了浅拷贝方法✌️。

## 深拷贝 deep clone
浅拷贝只是对对象的顶层属性进行了拷贝，而对于对象中嵌套的对象是没有进行拷贝的，拷贝对象和被拷贝对象中对应的属性值都是指向同一个（引用相同）嵌套对象。

深拷贝遇到嵌套的对象会递归进行拷贝，保证拷贝对象和被拷贝对象中没有属性值是引用同一对象，这样拷贝对象和被拷贝对象就都是单独的对象了。

显然进行浅拷贝要比深拷贝更高效率。

我们只需要对上一章节中的浅拷贝方法 `shallowCopy` 进行修改，检测对象属性中是否包含对象，若是，递归调用进行拷贝即可得到深拷贝方法。
```
function naiveDeepCopy( original ) {
    // First create an empty object with
    // same prototype of our original source
    var clone = Object.create( Object.getPrototypeOf( original ) ) ;

    var i , descriptor , keys = Object.getOwnPropertyNames( original ) ;

    for ( i = 0 ; i < keys.length ; i ++ ) {
        // Save the source's descriptor
        descriptor = Object.getOwnPropertyDescriptor( original , keys[ i ] ) ;

        if ( descriptor.value && typeof descriptor.value === 'object' )
        {
            // If the value is an object, recursively deepCopy() it
            descriptor.value = naiveDeepCopy( descriptor.value ) ;
        }

        Object.defineProperty( clone , keys[ i ] , descriptor ) ;
    }
    return clone ;
}
```

当然，这个目前这个深拷贝方法还存在着一些问题：
- 对象中存在的循环引用(Circular references)会导致调用栈溢出
- `Date` 和 `Array` 类型的对象可能不能正常拷贝
- 通过闭包作用域来实现私有成员的这类对象不能真正的被拷贝【Design pattern emulating private members using a closure's scope cannot be truly cloned (e.g. the revealing pattern)】

1、循环引用
```
var o = {
    a: 'a',
    sub: {
        b: 'b'
    },
    sub2: {
        c: 'c'
    }
} ;

o.loop = o ;
```
如上代码中，对象 `o` 有一个 `loop` 属性值执行自身，就构成了循环引用。此时通过上述的 `naiveDeepCopy` 方法深拷贝对象 `o`，由于没有循环引用的检测，就会 `o.loop.loop.loop.loop.loop...` 一直递归遍历下去，直到栈溢出。
所以在深拷贝方法中，必须对是否存在循环引用进行检测。

2、闭包作用域
看一下如下代码：
```
function myConstructor()
{
    var myPrivateVar = 'secret' ;

    return {
        myPublicVar: 'public!' ,
        getMyPrivateVar: function() {
            return myPrivateVar ;
        } ,
        setMyPrivateVar( value ) {
            myPrivateVar = value.toString() ;
        }
    };
}

var o = myContructor() ;
```
对象 `o` 有三个属性，一个是字符串，另外两个是方法。方法中用到一个变量 `myPrivateVar`，存在于 `myConstructor()` 的函数作用域中，当 `myConstructor` 构造函数调用时，就创建了这个变量 `myPrivateVar`，然而这个变量并不是通过构造函数创建的对象 `o` 的属性，但是它任然可以被这两个方法使用。

因此，如果尝试深拷贝对象 `o`，那么拷贝对象 `clone` 和被拷贝对象 `original` 中的方法都是引用相同的 `myPrivateVar` 变量。

但是，由于并没有方式改变闭包的作用域，所以这种模式创建的对象不能正常深拷贝是可以接受的。

最后，我们来看一下最终实现出来的深拷贝方法：
> 来自：[tree-kit library](https://github.com/cronvel/tree-kit)

```
clone( original , [circular] )
```
- **original**：Object the source object to clone
- **circular**：boolean (default to false) if true then circular references are checked and each identical objects are reconnected (referenced), if false then nested object are blindly cloned

该实现方法中自动检测是否存在循环引用（`circular`参数为 `true`），并自动处理其拷贝；同时对递归拷贝进行了优化，采用非递归的宽度优先遍历方式
> 关于遍历树，请参考: [JS 中遍历树](http://objcer.com/2017/02/26/traverse-the-tree/#more)

```
function clone( originalObject , circular ) {
    // First create an empty object with
    // same prototype of our original source
    var propertyIndex ,
        descriptor ,
        keys ,
        current ,
        nextSource ,
        indexOf ,
        copies = [ {
            source: originalObject ,
            target: Object.create( Object.getPrototypeOf( originalObject ) )
        } ] ,
        cloneObject = copies[ 0 ].target ,
        sourceReferences = [ originalObject ] ,
        targetReferences = [ cloneObject ] ;

    // First in, first out
    while ( current = copies.shift() )
    {
        keys = Object.getOwnPropertyNames( current.source ) ;

        for ( propertyIndex = 0 ; propertyIndex < keys.length ; propertyIndex ++ )
        {
            // Save the source's descriptor
            descriptor = Object.getOwnPropertyDescriptor( current.source , keys[ propertyIndex ] ) ;

            if ( ! descriptor.value || typeof descriptor.value !== 'object' )
            {
                Object.defineProperty( current.target , keys[ propertyIndex ] , descriptor ) ;
                continue ;
            }

            nextSource = descriptor.value ;
            descriptor.value = Array.isArray( nextSource ) ?
                [] :
                Object.create( Object.getPrototypeOf( nextSource ) ) ;

            if ( circular )
            {
                indexOf = sourceReferences.indexOf( nextSource ) ;

                if ( indexOf !== -1 )
                {
                    // The source is already referenced, just assign reference
                    descriptor.value = targetReferences[ indexOf ] ;
                    Object.defineProperty( current.target , keys[ propertyIndex ] , descriptor ) ;
                    continue ;
                }

                sourceReferences.push( nextSource ) ;
                targetReferences.push( descriptor.value ) ;
            }

            Object.defineProperty( current.target , keys[ propertyIndex ] , descriptor ) ;

            copies.push( { source: nextSource , target: descriptor.value } ) ;
        }
    }

    return cloneObject ;
} ;
```

## 其他
一、在 ES6 之前，JS 是没有提供对象拷贝的方法，但在 ES6 中有两种方式可以实现对象的**浅拷贝**：
1、`Object.assign`
```
const existing = { a: 1, b: 2, c: 3 };
const clone = Object.assign({}, existing);
```
2、Object rest/spread destructuring
```
const existing = { a: 1, b: 2, c: 3 };

const { ...clone } = existing;
```
> 这种方式可能浏览器支持并不是很好，在 babel 编译后的代码中我们注意到：
```
function _objectWithoutProperties(obj, keys) {
    var target = {};
    for (var i in obj) {
        if (keys.indexOf(i) >= 0) continue;
        if (!Object.prototype.hasOwnProperty.call(obj, i)) continue;
        target[i] = obj[i];
    }
    return target;
}
// 示例一:
// 编译前
const existing = { a: 1, b: 2, c: 3 };
const { ...clone } = existing;
// 编译后
var existing = { a: 1, b: 2, c: 3 };
var clone = _objectWithoutProperties(existing, []);

// 示例二:
// 编译前
const existing = { a: 1, b: 2, c: 3 };
const { a, ...clone } = existing;
// 编译后
var existing = { a: 1, b: 2, c: 3 };
var a = existing.a;
var clone = _objectWithoutProperties(existing, ['a']);
```

二、`JSON.parse` 和 `JSON.stringify`
如果一个对象可以被序列化成 JSON，那么可以通过 `JSON.parse` 和 `JSON.stringify` 深拷贝一个对象：
```
var existing = { a: 1, b: { c: 2 } };
var copy = JSON.parse(JSON.stringify(existing));
existing.b.c = 3; // copy.b.c will not change
console.log(copy.b.c) // 2
```
需要注意的是如果对象中存在属性值是 `Date` 的话，`JSON.stringify`序列化时会把 `Date` 对象转换成用 ISO 格式字符串表示，但是在 `JSON.parse` 反序列化时，却没有将这个 ISO 格式字符串转换回 `Date` 对象。
```
var existing = { a: new Date() };
var copy = JSON.parse(JSON.stringify(existing));

existing.a instanceof Date // true
copy.a instanceof Date // false
```

## 参考链接
[Understanding Object Cloning in Javascript - Part. I](http://blog.soulserv.net/understanding-object-cloning-in-javascript-part-i/)
[Understanding Object Cloning in Javascript - Part. II](http://blog.soulserv.net/understanding-object-cloning-in-javascript-part-ii/)


