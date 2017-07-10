title: 探究 emoji 字符长度
tags:
---
![](http://7vikhl.com1.z0.glb.clouddn.com/C0E1DEF1-900B-4372-B8CB-B166A1ABBA57.png)
我们注意到在 Twitter，微博，谷歌翻译的输入框中输入 emoji 字符，都不能正确的判断字符个数。🤷‍
在阅读本文之前，需要先对 Unicode 编码有所了解，参考: [JavaScript Unicode 编码那些事](http://objcer.com/2017/05/21/JavaScript-Unicode/)

<!-- more -->

## String length != Char count
```
'😂'.length // 2
'1️⃣'.length // 3
'👨‍👨‍👦'.length // 8
'👨‍👩‍👧‍👦'.length // 11
```

## 关于 Unicode
Unicode 编码范围是从 U+0000 到 U+10FFFF，每一个编码（也称之为码位 code point）表示一个 Unicode 字符；而这么多码位有划分成 17 个平面：
- 第一个平面(U+0000 ~ U+FFFF): 基本平面(Basic Multilingual Plane - *BMP*)
- 其他 16 个平面(U+100000 ~ U+10FFFF): 补充平面(Supplementary Planes)

其中 emoji 作为一种特殊编码在补充平面，例如：
```
'😀'.codePointAt(0).toString(16) // 0x1f600
'😂'.codePointAt(0).toString(16) // 0x1f602
```

## 关于 emoji
> An Emoji as we know it today **is defined by at least one code point** in the Unicode range. This means that there are also several Emoji out there being a combination of several different Emoji and code points. These combinations are called **sequences**.

一个 emoji 字符至少由一个码位表示，一个 emoji 有多个码位表示的我们称之为序列(sequences)。可以在 [Full Emoji List](http://unicode.org/emoji/charts/full-emoji-list.html) 页面查看到所有 emoji 的编码信息
![](http://7vikhl.com1.z0.glb.clouddn.com/5E88BF15-66B7-4A32-9125-B3E019C52E9C.png)

在 JavaScript 中 `String.length` 计算字符长度时，认为两个字节为一个字符，所以一个基本平面上的字符长度都为 1；而一个补充平面上的字符使用代理对，由四个字符表示，所以长度为 2；
```
// a: 0x61
'a'.lenght // 1

//💩: U+d83d U+dca9
'💩'.length // 2

// '1️⃣': U+0031 U+FE0F U+20E3
'1️⃣'.length // 3
```

### Modifier sequences
![](http://7vikhl.com1.z0.glb.clouddn.com/480D3F20-0262-40E4-B7C6-E068A9958692.png)
我们注意部分人类相关的 **Human emoji** 可以选择其他 5 种不同的肤色(**Skin color**)，默认颜色的 emoji 称之为 **neutral emoji**，不同肤色称之为 **modifier**，这些不同肤色的 emoji 我们称之为 **Modifier sequences**。
> These modifiers are called EMOJI MODIFIER FITZPATRICK TYPE-1-2, -3, -4, -5, and -6 (U+1F3FB–U+1F3FF): 🏻 🏼 🏽 🏾 🏿

Code point|	default| FITZ-1-2(U+1F3FB)| FITZ-3(U+1F3FC)| FITZ-4(U+1F3FD)| FITZ-5(U+1F3FE)| FITZ-6(U+1F3FF)
---|---|---|---|---|---|---
U+1F466: BOY|	👦|	👦🏻|	👦🏼|	👦🏽|	👦🏾|	👦🏿
U+1F467: GIRL|	👧|	👧🏻|	👧🏼|	👧🏽|	👧🏾|	👧🏿
U+1F468: MAN|	👨|	👨🏻|	👨🏼|	👨🏽|	👨🏾|	👨🏿
U+1F469: WOMAN| 👩|	👩🏻|	👩🏼|	👩🏽|	👩🏾|	👩🏿

```
// U+1F467 + U+1F3FD
👧 + 🏽
> 👧🏽
```
![](http://7vikhl.com1.z0.glb.clouddn.com/E3F5C223-E3E9-4ECC-AD67-DF09D6D62DD6.png)

明显的，这类 emoji 的通过 `String.length` 获取长度为 **4**

### ZWJ sequences
接下来里了解一下 **family emoji**，譬如我们常见的：三口之家：👪，四口之家：👨‍👩‍👧‍👦，等等；其中值得注意的是：
- 有一个 neutral family emoji (U+1F46A - ‍👪) 由一个码位表示
- 而其他 family emoji 都是 **Zero-Width-Joiner sequence**；它们由多个 emoji 和 zero-width-joiner (U+200D) 组合而成

注：zero-width-joiner (U+200D) 这个码位就如同胶水一般将不同的 emoji 组合成一个 emoji；zero-width-joiner (U+200D) 只占一个长度
```
// neutral family
// U+1F46A
// length: 2
> 👪

// ZWJ sequence: family (man, woman, boy)
// U+1F468 + U+200D + U+1F469 + U+200D + U+1F466
// 👨‍ + U+200D + 👩‍ + U+200D + 👦
// length: 8
> ‍👨‍👩‍👦

// ZWJ sequence: family (woman, woman, girl)
// U+1F469 + U+200D + U+1F469 + U+200D + U+1F467
// 👩‍ + U+200D + 👩‍ U+200D + 👧
// length: 8
> ‍👩‍👩‍👧

// ZWJ sequence: family (woman, woman, girl, girl)
// U+1F469 + U+200D + U+1F469 + U+200D + U+1F467 + U+200D + U+1F467
// 👩‍ + U+200D + 👩‍ + U+200D + 👧‍ + U+200D + 👧
// length: 11
> ‍👩‍👩‍👧‍👧
```

拆分，组合 family emoji：
```
[...'👨‍👩‍👦']
>  ["👨", "‍", "👩", "‍", "👦"]

var family =  ["👨", "‍", "👩", "‍", "👦"]
console.log(family.join(''))
> "👨‍👩‍👦"
```

## 解决字符长度终极解决方案
```
```

## 参考链接
[JavaScript has a Unicode problem](https://mathiasbynens.be/notes/javascript-unicode)
[Emoji.prototype.length — a tale of characters in Unicode](https://www.contentful.com/blog/2016/12/06/unicode-javascript-and-the-emoji-family/)