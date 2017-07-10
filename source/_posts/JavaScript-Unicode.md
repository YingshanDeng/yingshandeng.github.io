title: JavaScript Unicode 编码那些事
date: 2017-05-21 16:36:57
tags: [BMP, charCodeAt, codePointAt]
categories: Unicode
---


![](http://7vikhl.com1.z0.glb.clouddn.com/IMG_1915.PNG)
<!-- more -->

## BMP(Basic Multilingual Plane)字符

Unicode 是目前绝大多数程序使用的字符编码。Unicode 标识符通过一个明确的名字和一个整数来作为它的码点/码位 (code point)。比如，“©️” 字符可以用“版权标志” 和码位 U+00A9 (0xA9，也可以写作十进制 169) 来表示。

**码点/码位**是为每一个字符提供一个全局唯一的标识符，一个码位映射一个字符，码位值的范围是从U+0000到U+10FFFF，可以表示超过110万个符号。

Unicode 字符分为 17 组平面，每个平面拥有 2^16 (65,536) 个码位。每一个码位都可以用 16 进制 xy0000 到 xyFFFF 来表示，这里的 xy 是表示一个 16 进制的值，从 00 到 10。

而当 xy 是 00 (码点范围是从U+0000到U+FFFF) 的时候，也就是 Unicode 最前 2^16 (65,536) 个字符，被称为**基本平面 BMP(Basic Multilingual Plane)**，最常见的字符都在这个平面上，这也是 Unicode 最先定义和最先公布的一个平面。

其余 16 个平面（U+100000 到 U+10FFFF）称为**补充平面(supplementary planes, or astral planes)**，也称之为补充字符，相对于 BMP 字符而言，这些字符称之为非 BMP 字符。要区分是非 BMP 字符很简单：其码位需要超过 4 位 16 进制表示

## UTF-16 和 UCS-2

### UTF-16
UTF-16 对于 BMP 字符的码位，用 2 个字节进行编码；而非 BMP 字符的码位，用 4 个字节组成代理对（surrogate pair）来表示。

关于**代理对**：前两个字节称为高位代理或者顶部代理，**范围在 0xD800 到 0xDBFF 之间**；后两个字节称为低位代理或者尾部代理，**范围在 0xDC00 到 0xDFFF 之间**。

**码位（code points）和代理对（surrogate pairs）之间的转换：**
假设：一个码位 C 大于 0xFFFF 的编码使用代理对 <H, L> 来表示的公式为：
```
H = Math.floor((C - 0x10000) / 0x400) + 0xD800
L = (C - 0x10000) % 0x400 + 0xDC00
```
转换公式变换后，比如从代理对 <H, L> 转换成一个 Unicode 码位 C，可以使用公式：
```
C = (H - 0xD800) * 0x400 + L - 0xDC00 + 0x10000
```

代码实现如下：
```
// codePoint > 0xFFFF
function getSurrogates(codePoint) {
  var high = Math.floor((codePoint - 0x10000) / 0x400) + 0xD800;
  var low = (codePoint - 0x10000) % 0x400 + 0xDC00;
  return [high, low];
}

function getCodePoint(high, low) {
  var codePoint = (high - 0xD800) * 0x400 + low - 0xDC00 + 0x10000;
  return codePoint;
}
```

### UCS-2
UCS(Universal Character Set) 通用字符集，是一个 ISO 标准，UCS-2 用 2 个字节表示 BMP 字符的码点，UCS-2 是一个过时的编码方式，因为它只能编码基本平面 BMP 的码点，在 BMP 字符的编码上，与 UTF-16 是一致的，所以可以认为是UTF-16的一个子集。

### JavaScript 是用哪一种编码方式的呢？
那么 JavaScript 是用哪一种编码方式的呢？UTF-16 还是 UCS-2 呢？答案是：UCS-2
基于如下年表：
- 1990 UCS-2 诞生
- 1995.5 JavaScript 诞生
- 1996.7 UTF-16 诞生

所以 JS 诞生之时 UTF-16 还没有问世，所以只能用 来处理字符，而这也为字符处理留下了隐患，而后来通过不断完善，譬如引入 UTF-16 将问题一步步解决。

## JavaScript 字符处理

### length 属性
```
var char = '💩'
char.length === 2 // true

char.charCodeAt(0).toString(16) // d83d
char.charCodeAt(1).toString(16) // dca9
char === '\ud83d\udca9' // true
```
**分析：**
由于 💩 这个 emoji 在 JS中编码为 `\ud83d\udca9`，而 JS 认为每两个字节即表示一个字符，所以 💩 这个 emoji 字符的字符长度就为 2。所以在输入框长度限制时就会有问题，譬如：输入长度限制 20，但是输入 10 个 emoji 后，字符串长度就达到 20 了。

**解决方法：**
上一节中，了解到 UTF-16 使用代理对，通过四个字节来表示非 BMP 字符，前两个字节和后两个字节都有各自的范围，所以：
```
var regexAstralSymbols = /[\uD800-\uDBFF][\uDC00-\uDFFF]/g; // 匹配UTF-16的代理对

function countSymbols(string) {
    // 把代理对改为一个BMP的字符，这时候取长度就正常
    return string.replace(regexAstralSymbols, '_').length;
}
countSymbols('💩'); // 1
```

### 反转字符串
对于反转字符串，我们可以很快的写出如下函数：
```
function reverse(str) {
    return str.split('').reverse().join('');
}
```
进行一下测试：
```
reverse('123') // "321"
reverse('💩') // "��"
```
**分析：**
� 的 Unicode 码位是 U+FFFD，通常用来表示 Unicode 转换时无法识别的字符（也就是乱码）。当💩（\ud83d\udca9）通过上述方法反转时，变成\udca9\ud83d，不是一个合法的代理对（高低字节范围不同），同时 Unicode 规定代理对范围内的码位不能单独出现，所以 js 只能用 � 表示了。

**解决方法：**
ES6 中数组新增了静态方法 `Array.from`，此方法支持对代理对的解析
```
function reverse(string) {
    return Array.from(string).reverse().join('');
}
```

> `Array.from` 方法的作用是将一个 `array-like` 或者 `iterable object` 转换成一个 Array 对象。最常见的是 `NodeList` 和函数参数 `arguments`，在实际开发过程中，需要将它们转换成数组：
```
// 以往的解决方法：
var imgs = [].slice.call(document.querySelectorAll('img'));
var args = [].slice.call(arguments);

// 现在有了 `Array.from` 我们就可以这样：
var imgs = Array.from(document.querySelectorAll('img'));
var args = Array.from(arguments);
```

### 关于 `charCodeAt` 和 `codePointAt`

> [Mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)文档说明： The **charCodeAt()** method returns an integer between 0 and 65535 representing the UTF-16 code unit at the given index (the UTF-16 code unit matches the Unicode code point for code points representable in a single UTF-16 code unit, but might also be the first code unit of a surrogate pair for code points not representable in a single UTF-16 code unit, e.g. Unicode code points > 0x10000). If you want the entire code point value, use **codePointAt()**.

理解一下：UTF-16 编码方式对于 BMP 字符，使用两个字节就可以表示，这两个字节表示一个 `code unit`；对于非 BMP 字符，则需要 4 个字节组成的代理对来表示，这四个字节则表示两个 `code unit`。所以一个码位 (code point) 由一个或者两个 `code unit` 组成
- `charCodeAt` 方法接收一个 `index` 参数，返回指定位置上的 `code unit` 的整数值，范围是 0 到 65535。
```
var char = 'a'; // BMP 字符
char.charCodeAt(0).toString(16); // 61
char.charCodeAt(1).toString(16); // NaN

var char = '💩'; // 非BMP 字符
char.charCodeAt(0).toString(16); // d83d
char.charCodeAt(1).toString(16); // dca9
```
- `codePointAt` 方法同样接收一个 `index` 参数，返回从指定位置开始，整个码位(code point)的值。
```
var char = 'a'; // BMP 字符
char.codePointAt(0).toString(16); // 61
char.codePointAt(1).toString(16); // NaN

var char = '💩'; // 非BMP 字符
char.codePointAt(0).toString(16); // 1f4a9
char.codePointAt(1).toString(16); // dca9
```

通过上述的代码示例中，我们发现：
对于 BMP 字符，`charCodeAt` 和 `codePointAt` 的行为一致，都是返回整个字符的码位 (code point)；
对于非 BMP 字符，`charCodeAt` 通过指定参数，返回不同位置上的 `code unit` 值；而 `codePointAt` 当参数为 0 时，返回整个字符的码位 (code point)，当参数为 1 时，返回的是低位代理 `code unit` 的值。

通过 `codePointAt` 方法，我们可以判断一个字符是两个字节(16位)表示，还是四个字节(32位)表示：
```
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}

is32Bit('A') // false
is32Bit('💩') // true
```

### 码位和字符转换
`charCodeAt` 方法可以获取 BMP 字符的码位；`codePointAt` 方法可以获取 BMP 和非 BMP 字符的码位。反之，如何将一个码位转换成字符呢？

`charCodeAt` 和 `codePointAt` 对应着的有 `fromCharCode` 和 `fromCodePoint` 这两个方法，同样的，`fromCharCode` 只能讲一个 BMP 字符的码位转换回字符，对于非 BMP 字符的码位就无能为力了，得使用 `fromCodePoint` 方法。

```
var c = 'a';
var d = '💩';
String.fromCharCode(c.charCodeAt(0)) // a
String.fromCharCode(d.charCodeAt(0)) // �
String.fromCodePoint(d.codePointAt(0)) // 💩
```

在 ES5 中允许通过 `\u` 加上码位组成的编码序列来表示 16 位的 Unicode BMP 字符，即：`\uXXXX`。例如：`\u0061` 表示字符 `a`
```
console.log('\u0061') // a
```
当然这只是对于 BMP 字符有效，对于非 BMP 字符，码位范围大于 `OxFFFF` 时，这种表示方式就有问题了，例如
```
console.log('\u1f4a9') // Ὂ9
```
`1f4a9` 是 emoji 💩 的码位，因为 Unicode 的编码序列总是包含 16 位，所以 JS 会把 `\u1f4a9` 分成两个字符 `\u1f4a` 和 `9`，所以最终输出：`Ὂ9` 两个字符。

ES6 通过扩展 Unicode 的编码序列解决了这个问题，将码位置于花括号内，即：`\u{XXXXX}`，可表示任何字符的码位。
```
console.log('\u{0061}') // a
console.log('\u{1f4a9}') // 💩
```

### 正则匹配 `u` 修饰符
正则表达式也是基于两个字节(16位)，一个 code unit 来表示单个字符的，那么对于非 BMP 字符，例如：emoji 💩 就会被认为是两个字符，在正则匹配中就会遇到问题。例如：
```
var char = '💩';
console.log(char.length) // 2
console.log(/^.$/.test(char)); // false
```
在正则中，`.` 小数点，是用于匹配除换行符之外的任何单个字符，但是对于非 BMP 字符 💩 JS 把它认为是两个字符，所以匹配失败。
ES6 为正则表达式定义了一个新的修饰符，`u` 也即 Unicode。当一个正则表达式设置了 `u` 修饰符时，它将切换模式为工作在字符串，而不是 code unit。这意味着正则表达式在包含非 BMP 字符的字符串中不会迷惑。
```
console.log(/^.$/u.test(char)); // true
```

## 参考链接
[每天学点ES6－字符串](http://cookfront.github.io/2015/06/04/es6-string/)
[Javascript有个Unicode的天坑](http://www.alloyteam.com/2016/12/javascript-has-a-unicode-sinkhole/)