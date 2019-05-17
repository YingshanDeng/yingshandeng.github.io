title: 学习正则表达式
date: 2018-01-22 21:26:08
tags: [正则表达式]
categories: JS
---

![](http://cdn.objcer.com/regex.jpg)
对正则表达式，我们并不陌生。在很多地方都会用到，尤其是字符串处理，正则表达式更是一把利器。本文通过一个例子来学习总结一下正则表达式的一些知识内容 🤓

<!-- more -->

## 问题引入
`index.html` 页面中，有如下一段代码：
```
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <meta name="theme-color" content="#000000">
  <title>React SharedPen App</title>
  <script src="./pace/pace.min.js"></script>
  <link rel="stylesheet" href="./pace/pace-theme-minimal.css">

  <!-- SharedPen scripts -->
  <!-- js:./dist/sharedpen.min.js -->
  <script src="http://localhost:5000/Utils.js"></script>
  ...
  <script src="http://localhost:5000/SharedPen.js"></script>
  <!-- end -->

  <!-- SharedPen stylesheet -->
  <!-- css:./dist/SharedPen.css -->
  <link rel="stylesheet" href="http://localhost:5000/SharedPen.css">
  <!-- end -->
</head>
```
注意这两段注释标签
```
<!-- js:./dist/sharedpen.min.js -->
...
<!-- end -->

<!-- css:./dist/SharedPen.css -->
...
<!-- end -->
```
在页面在开发环境中，加载的是本地源文件，在生产环境中，我希望将其替换成压缩文件：`./dist/sharedpen.min.js` 和 `./dist/SharedPen.css`。

**解决思路是：**使用 webpack 插件 [html-replace-webpack-plugin](https://www.npmjs.com/package/html-replace-webpack-plugin) 配置正则表达式，然后进行替换。最终的代码如下:
```
const HtmlReplaceWebpackPlugin = require('html-replace-webpack-plugin')
const tpl = {
  css: '<link rel="stylesheet" type="text/css" href="%s">',
  js: '<script type="text/javascript" src="%s"></script>'
}

module.exports = {
  // Definition for Webpack plugins
  plugin: [
    // Replace html contents with string or regex patterns
    new HtmlReplaceWebpackPlugin([{
      pattern: /(<!--\s*)(css|js):([\w-\/\.]+)(\s*-->)([\s\S]*?)(<!--\s*end\s*-->)/,
      replacement: function (match, $1, type, file) {
        switch (type) {
          case 'css':
          case 'js':
            return tpl[type].replace('%s', file)
          default:
            return ''
        }
      }
    }])
  ]
}
```

其中关键就是写出正则表达式，匹配出那两段注释代码。正则语法可以参考文末附录二，为了方便测试，我们可以在 https://regex101.com/ 这个网站上编写正则表达式。

## 撰写正则表达式
```
<!-- js:./dist/sharedpen.min.js -->
...
<!-- end -->

<!-- css:./dist/SharedPen.css -->
...
<!-- end -->
```

### 分成三部分处理
观察这两段注释，每一段可以分成三部分，根据正则语法，可以写出如下三部分的正则表达式：
- 起始部分 匹配 `<!-- js:./dist/sharedpen.min.js -->`
  ```
  <!--\s*(css|js):[\w\-\/\.]+\s*-->
  ```
- 中间内容部分
  ```
  [\s\S]*
  ```
- 结束部分 匹配 `<!-- end -->`
  ```
  <!--\s*end\s*-->
  ```

### 子表达式
在第一部分 `<!-- js:./dist/sharedpen.min.js -->`，我们还需要正则表达式能匹配出：文件类型和文件路径

> 注意到
1、`(pattern)` **匹配 pattern 并捕获该匹配的子表达式。可以使用 $1…$9 属性从结果“匹配”集合中检索捕获的匹配。**
2、在 JavaScript 中 `$1…$9` 属性是包含括号子串匹配的正则表达式的静态只读属性，我们可以通过 `RegExp.$1, ..., RegExp.$9` 来使用它们

通过括号写出子表达式，匹配出我们需要的内容
```
<!--\s*(css|js):([\w\-\/\.]+)\s*-->
```

```
var regexp = new RegExp(/<!--\s*(css|js):([\w\-\/\.]+)\s*-->/)
var str = '<!-- js:./dist/sharedpen.min.js -->'
regexp.exec(str)
console.log(RegExp.$1) // "js"
console.log(RegExp.$2) // "./dist/sharedpen.min.js"
```

### 最长匹配（贪婪匹配）和最短匹配（懒惰匹配）
到目前为止，我们的正则表达式为：`<!--\s*(css|js):([\w\-\/\.]+)\s*-->[\s\S]*<!--\s*end\s*-->`，但是会发现，此时匹配有点问题：
![](http://cdn.objcer.com/51E1FD12-5B3D-4600-8EEE-C00AE4485F46.png)

我们是希望遇到第一个 `<!-- end -->` 就结束匹配，而现在却是匹配到最后一个。这个时候就需要了解正则表达式的最长匹配（贪婪匹配）和最短匹配（懒惰匹配）
例子一：
```
var str = 'abc123def'

var regexp1 = new RegExp(/abc[\d]*/) // 最长匹配
var regexp2 = new RegExp(/abc[\d]*?/) // 最短匹配

console.log(regexp1.exec(str)[0]) // "abc123"
console.log(regexp2.exec(str)[0]) // "abc"
```

例子二：
```
var str = '/*** 注释1 ****/ var name = "aty"; /***注释2****/'
var regexp = new RegExp(/\/\*([\s\S]*?)\*\//)

console.log(regexp.exec(str)[0]) // 使用 `*?` 就可以正确的匹配到注释内容
```

我们会发现，最长和最短匹配的差别是，前者匹配尽可能多的字符，后者匹配尽可能少的字符。一般正则表达式引擎默认都是最长匹配，**如果想要最短匹配，那么需要在数量修饰符(例如：`*`，`+`)后面添加一个 `?` 变成最短匹配**

所以此时执行对我们的正则表达式增加一个 `?` 即可
```
<!--\s*(css|js):([\w\-\/\.]+)\s*-->[\s\S]*?<!--\s*end\s*-->
```

### 全局匹配
JavaScript 中使用正则表达式有两种方式：
```
# 直接量语法
/pattern/attributes

# 创建 RegExp 对象的语法
new RegExp(pattern, attributes);
```
**参数:**
- pattern 是一个字符串，指定了正则表达式
- attributes 是一个可选的字符串，包含属性: **"g"、"i" 和 "m"，分别用于指定全局匹配、区分大小写的匹配和多行匹配**。

其中 `g` 指定正则表达式全局匹配，那么有什么特殊之处呢？
```
var str = 'abc123def'
var regexp = new RegExp(/abc[\d]*/, 'g')

console.log(regexp.test(str)) // true
console.log(regexp.test(str)) // false
console.log(regexp.test(str)) // true
```

其中第二次 console 输出为啥是 `false` 呢？

- 在全局匹配模式下可以对指定要查找的字符串**执行多次匹配**。
- 每次匹配使用当前正则对象的 `lastIndex` 属性的值作为在目标字符串中开始查找的起始位置。
- `lastIndex` 属性的初始值为 0，找到匹配的项后，`lastIndex` 的值被重置为匹配内容的下一个字符在字符串中的位置索引，用来标识下次执行匹配时开始查找的位置。
- 如果找不到匹配的项，`lastIndex` 的值会被设置为 0。
- 没有设置正则对象的全局匹配标志时 `lastIndex` 属性的值始终为0，每次执行匹配仅查找字符串中第一个匹配的项。

如果我们每次执行 `regexp.test(str)` 方法后，查看一下正则对象的 lastIndex 属性的值，就清楚了。
```
var str = 'abc123def'
var regexp = new RegExp(/abc[\d]*/, 'g')

console.log(regexp.test(str)) // true
console.log(regexp.lastIndex) // 6

console.log(regexp.test(str)) // false
console.log(regexp.lastIndex) // 0

console.log(regexp.test(str)) // true
console.log(regexp.lastIndex) // 6
```

第一次执行 `regexp.test(str)` 前，正则对象的 `lastIndex` 属性值为 0，匹配到字符 `abc123` 后，正则对象的 `lastIndex` 属性值变成 6；
在第二次执行 `regexp.test(str)` 时，从第 7 个字符(从0开始)开始进行匹配，当然就匹配失败啦。

下面再来看看正则对象的另一个方法 `exec` 在全局模式下的例子：
```
var str = `
<!-- js:./dist/sharedpen.min.js -->
...
<!-- end -->
<!-- css:./dist/SharedPen.css -->
...
<!-- end -->`

var regexp = new RegExp(/<!--\s*(css|js):([\w\-\/\.]+)\s*-->[\s\S]*?<!--\s*end\s*-->/g)
var matches
while((matches = regexp.exec(str)) != null) {
  str = str.replace(matches[0], '<div>test</div>')
}
console.log(str)
```

我们希望代码执行匹配替换后，输出结果是：
```
<div>test</div>
<div>test</div>
```

但是实际上输出结果是：
```
<div>test</div>
<!-- css:./dist/SharedPen.css -->
...
<!-- end -->
```

显然是第二次匹配替换出问题了。有了前面的经验，我们很快就知道，是全局模式匹配的问题，第一次全局匹配后，正则对象的 `lastIndex` 属性值发生变化，影响了第二次正则匹配，我们只需去掉全局匹配 `g` 即可解决问题。


🏗 至此，我们终于可以得到如下的正则表达式：
```
/(<!--\s*)(css|js):([\w-\/\.]+)(\s*-->)([\s\S]*?)(<!--\s*end\s*-->)/
```

------

## [附录一] 常用方法
RegExp 对象方法：`test`, `exec`

用法 | 说明 | 返回值
--- | --- | ---
`RegExpObj.test(string)` | 用于检测一个字符串是否匹配某个模式 | 匹配返回 true，否则返回 false
`RegExpObj.exec(string)` | 用于检索字符串中的正则表达式的匹配 |  返回一个数组，其中存放匹配的结果。如果未找到匹配，则返回值为 null

String 对象方法：`match`, `replace`

用法 | 说明 | 返回值
--- | --- | ---
`stringObj.match( regexp/substr )` | 在字符串内检索指定的值，或找到一个或多个正则表达式的匹配 | 返回一个数组，其中存放匹配的结果。如果未找到匹配，则返回值为 null
`stringObj.replace( regexp/substr, replacement )` | 根据 regexp/substr 进行正则匹配,把匹配结果替换为 replacement | 一个新的字符串

## [附录二] 正则表达式语法
学习正则表达式，首先要了解其语法规则，以下规则来自：**MSDN 正则表达式语法:** https://msdn.microsoft.com/zh-cn/library/ae5bf541(v=vs.90).aspx

字符 | 说明
---|---
`\` | **将下一字符标记为特殊字符、文本、反向引用或八进制转义符。**例如，“n” 匹配字符 “n”。“\n” 匹配换行符。序列 “\\” 匹配 “\”，“\(” 匹配 “(”。
`^` | **匹配输入字符串开始的位置。**如果设置了 RegExp 对象的 Multiline 属性，^ 还会与 “\n” 或 “\r” 之后的位置匹配。
`$` | **匹配输入字符串结尾的位置。**如果设置了 RegExp 对象的 Multiline 属性，$ 还会与 “\n” 或 “\r” 之前的位置匹配。
`*` | **零次或多次匹配前面的字符或子表达式。**例如，zo* 匹配 “z” 和 “zoo”。* 等效于 {0,}。
`+` | **一次或多次匹配前面的字符或子表达式。**例如，“zo+” 与 “zo” 和 “zoo” 匹配，但与 “z” 不匹配。+ 等效于 {1,}。
`?` | **零次或一次匹配前面的字符或子表达式。**例如，“do(es)?” 匹配 “do” 或 “does” 中的 “do”。? 等效于 {0,1}。
`{n}` | **n 是非负整数。正好匹配 n 次。**例如，“o{2}” 与 “Bob” 中的 “o” 不匹配，但与 “food” 中的两个“o”匹配。
`{n,}` | **n 是非负整数。至少匹配 n 次。**例如，“o{2,}” 不匹配 “Bob” 中的 “o”，而匹配 “foooood” 中的所有 o。“o{1,}” 等效于 “o+”。“o{0,}” 等效于 “o*”。
`{n,m}` | m 和 n 是非负整数，其中 n <= m。**匹配至少 n 次，至多 m 次。**例如，“o{1,3}” 匹配 “fooooood” 中的头三个 o。'o{0,1}' 等效于 'o?'。注意：您不能将空格插入逗号和数字之间。
`?` | **当此字符紧随任何其他限定符（*、+、?、{n}、{n,}、{n,m}）之后时，匹配模式是“非贪心的”。“非贪心的”模式匹配搜索到的、尽可能短的字符串，而默认的“贪心的”模式匹配搜索到的、尽可能长的字符串。**例如，在字符串 “oooo” 中，“o+?” 只匹配单个 “o”，而 “o+” 匹配所有 “o”。
`.` | **匹配除“\n”之外的任何单个字符。**若要匹配包括 “\n” 在内的任意字符，请使用诸如 “[\s\S]” 之类的模式。
`(pattern)` | **匹配 pattern 并捕获该匹配的子表达式。可以使用 $1…$9 属性从结果“匹配”集合中检索捕获的匹配。**若要匹配括号字符 ( )，请使用“\(”或者“\)”。
`(?:pattern)` | 匹配 pattern 但不捕获该匹配的子表达式，即它是一个非捕获匹配，不存储供以后使用的匹配。这对于用 “or” 字符 `(│)` 组合模式部件的情况很有用。例如，`industr(?:y│ies)` 是比 `industry│industries` 更经济的表达式。
`(?=pattern)` | 执行正向预测先行搜索的子表达式，该表达式匹配处于匹配 pattern 的字符串的起始点的字符串。它是一个非捕获匹配，即不能捕获供以后使用的匹配。例如，'Windows (?=95│98│NT│2000)' 匹配“Windows 2000”中的“Windows”，但不匹配“Windows 3.1”中的“Windows”。预测先行不占用字符，即发生匹配后，下一匹配的搜索紧随上一匹配之后，而不是在组成预测先行的字符后。
`(?!pattern)` | 执行反向预测先行搜索的子表达式，该表达式匹配不处于匹配 pattern 的字符串的起始点的搜索字符串。它是一个非捕获匹配，即不能捕获供以后使用的匹配。例如，'Windows (?!95│98│NT│2000)' 匹配“Windows 3.1”中的 “Windows”，但不匹配“Windows 2000”中的“Windows”。预测先行不占用字符，即发生匹配后，下一匹配的搜索紧随上一匹配之后，而不是在组成预测先行的字符后。
`x│y` | **匹配 x 或 y。**例如，'z│food' 匹配“z”或“food”。'(z│f)ood' 匹配“zood”或“food”。
`[xyz]` | 字符集。匹配包含的任一字符。例如，“[abc]”匹配“plain”中的“a”。
`[^xyz]` | 反向字符集。匹配未包含的任何字符。例如，“[^abc]”匹配“plain”中的“p”。
`[a-z]` | **字符范围。**匹配指定范围内的任何字符。例如，“[a-z]”匹配“a”到“z”范围内的任何小写字母。
`[^a-z]` | **反向范围字符。**匹配不在指定的范围内的任何字符。例如，“[^a-z]”匹配任何不在“a”到“z”范围内的任何字符。
`\b` | 匹配一个字边界，即字与空格间的位置。例如，“er\b”匹配“never”中的“er”，但不匹配“verb”中的“er”。
`\B` | 非字边界匹配。“er\B”匹配“verb”中的“er”，但不匹配“never”中的“er”。
`\cx` | 匹配 x 指示的控制字符。例如，\cM 匹配 Control-M 或回车符。x 的值必须在 A-Z 或 a-z 之间。如果不是这样，则假定 c 就是“c”字符本身。
`\d` | **数字字符匹配。**等效于 [0-9]。
`\D` | **非数字字符匹配。**等效于 [^0-9]。
`\f` | 换页符匹配。等效于 \x0c 和 \cL。
`\n` | 换行符匹配。等效于 \x0a 和 \cJ。
`\r` | 匹配一个回车符。等效于 \x0d 和 \cM。
`\s` | **匹配任何空白字符，包括空格、制表符、换页符等。**与 [ \f\n\r\t\v] 等效。
`\S` | **匹配任何非空白字符。**与 [^ \f\n\r\t\v] 等效。
`\t` | 制表符匹配。与 \x09 和 \cI 等效。
`\v` | 垂直制表符匹配。与 \x0b 和 \cK 等效。
`\w` | 匹配任何字类字符，包括下划线。与“[A-Za-z0-9_]”等效。
`\W` | 与任何非单词字符匹配。与“[^A-Za-z0-9_]”等效。
`\xn` | 匹配 n，此处的 n 是一个十六进制转义码。十六进制转义码必须正好是两位数长。例如，“\x41”匹配“A”。“\x041”与“\x04”&“1”等效。允许在正则表达式中使用 ASCII 代码。
`\num` | 匹配 num，此处的 num 是一个正整数。到捕获匹配的反向引用。例如，“(.)\1”匹配两个连续的相同字符。
`\n` | 标识一个八进制转义码或反向引用。如果 \n 前面至少有 n 个捕获子表达式，那么 n 是反向引用。否则，如果 n 是八进制数 (0-7)，那么 n 是八进制转义码。
`\nm` | 标识一个八进制转义码或反向引用。如果 \nm 前面至少有 nm 个捕获子表达式，那么 nm 是反向引用。如果 \nm 前面至少有 n 个捕获，则 n 是反向引用，后面跟有字符 m。如果两种前面的情况都不存在，则 \nm 匹配八进制值 nm，其中 n 和 m 是八进制数字 (0-7)。
`\nml` | 当 n 是八进制数 (0-3)，m 和 l 是八进制数 (0-7) 时，匹配八进制转义码 nml。
`\un` | 匹配 n，其中 n 是以四位十六进制数表示的 Unicode 字符。例如，\u00A9 匹配版权符号 (©)。

正则表达式从左往右进行计算，运算符的优先级顺序如下：

运算符 | 说明
--- | ---
`\` | 转义符
`(), (?:), (?=), []` | 括号和中括号
`*, +, ?, {n}, {n,}, {n,m}` | 限定符
`^, $, \ 任何元字符、任何字符` | 定位点和序列
│ | 替换

## 参考链接
[JavaScript RegExp 对象](http://www.w3school.com.cn/jsref/jsref_obj_regexp.asp)
[JS 进阶 test, exec, match, replace](https://segmentfault.com/a/1190000003497780)
[正则匹配在线网站](https://regex101.com/)
