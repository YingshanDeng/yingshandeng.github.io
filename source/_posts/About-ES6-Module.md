title: 关于 ES6 Module
date: 2017-04-19 20:44:58
tags: [Module, Babel, exports, module.exports]
categories: JS
---

*问题引入：*最近在 Babel 编译时遇到一个问题，对 ES6 Module 编译的问题，本文将对解决该问题过程中关于 ES6 Module 有关内容进行记录和介绍。

<!-- more -->

## 关于：CommonJS, AMD, CMD, UMD
- CommonJS: 用于服务端，模块同步加载，node.js 的模块系统就是基于 CommonJS 规范的实现
- AMD (Asynchronous Module Definition): 用于浏览器端，模块异步加载，目前，流行的 require.js 库就实现了 AMD 规范
- CMD (Common Module Definition): 是 SeaJS 在推广过程中对模块定义的规范化，和 AMD 有些类似，参考：[与 RequireJS 的异同](https://github.com/seajs/seajs/issues/277)
- UMD (Universal Module Definition): 兼容了 AMD 和 CommonJS，同时还支持老式的“全局”变量规范，使得模块化代码可以在前端浏览器和后台服务端中运行

## 问题阐述及解决
在 `test.js` 文件中，有如下一行代码：
```
export default 23;
```

在 `gulpfile.js` 文件中，有如下任务对其进行编译：
```
const gulp = require('gulp')
const $ = require('gulp-load-plugins')();

gulp.task('babel', function() {
    return gulp.src(['src/test.js'])
        .pipe($.babel({
            presets: ['es2015']
        }))
        .pipe(gulp.dest('bin/'));
});
```

执行 `gulp babel` 编译后，结果为：
```
"use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});

exports.default = 23;
```

那么使用过程中会遇到什么问题呢？
例如，我们在 `node.js` 文件中有如下代码：
```
var a = require('./test.js'); // { default: 23 }
console.log('---', a);
```
执行 `node node.js` 后，输出的结果为：`--- { default: 23 }`
如果想要 `a` 即为 23，那么需要：
```
var a = require('./test.js').default;
console.log('---', a);
```
显然每次 `require` 使用的时候需要手动添加 `default` 比较麻烦，而且代码也不好看🙃。那么有什么方法解决这个问题么？

**👇下面我们需要对 `exports` 和 `module.exports` 进行了解**

在 Node.js modules 中，我们对 `exports` 都不陌生，我们可以通过 `exports` 暴露任何合法的 JavaScript 对象，包括：boolean, number, date, JSON, string, function, array 等等。
```
exports.name = function() {
    console.log('My name is XiaoDeng');
};
```
然后在其他文件中使用：
```
var myName = require('./test.js');
myName.name(); // 'My name is XiaoDeng'
```
那么 `module.exports` 又是什么呢❓
> The `module.exports` object is created by the Module system.

所以真实存在，且最终返回给调用者的是 `module.exports`；而 `exports` 只是 `module.exports` 的 **shortcut/alias** 辅助别名。我们可以这么理解：
```
var module = new Module(...);
var exports = module.exports;
```
![](https://raw.githubusercontent.com/yingshandeng/image-host/master/data/exports&module.exports.png)
`module.exports` 初始值为一个空对象 `{}`; `exports` 是指向的对象和 `module.exports` 执向的对象相同。

`exports` 的作用在于收集属性和方法，例如：
```
exports.a = function() {
    console.log("a");
}
exports.b = function() {
    console.log("b");
}
```
此时，`exports` 和 `module.exports` 都是指向同一对象：{ a: [Function], b: [Function] }；
但是如果：
```
exports = function Something() {
    console.log('bla bla');
}
```
此时，`exports` 执向的对象和  指向的对象已经不同，此时 `exports` 执行的是一个函数，而 `module.exports` 却依然执向一个空对象 `{}`，二者的引用不同。可以通过 `exports = module.exports` 让 `exports` 重新指向 `module.exports` 即可保持二者引用相同。
> 对上述问题的理解就涉及到 [JS中是 pass by value 还是 pass by reference](http://objcer.com/2017/02/26/js-pass-by-value-or-by-reference/)

通过以上的分析，我们推荐是使用 `module.exports` 而非 `exports`。（Node.js文档中也是这么推荐的）

由此，我们可以通过添加 `module.exports` 来解决 `require` 时需要指定 `default` 的问题
```
'use strict';
Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.default = 'foo';
module.exports = exports['default'];
```
如上这个过程可以通过 **🚀[babel-plugin-add-module-exports](https://github.com/59naga/babel-plugin-add-module-exports)** 插件，在 babel 过程中自动处理。

由于浏览器中缺少 `module` `exports` `require` `global` 这四个 NodeJS 环境变量，所以不兼容 CommonJS，也就是说，浏览器中并不能使用上述的模块，那么为了解决如上问题，我们需要另一个插件 **🚀[ES2015 modules to UMD transform](http://babeljs.io/docs/plugins/transform-es2015-modules-umd/)**，将模块代码编译成 UMD 模式。

综上：下载插件，在 babel 过程中传入两个插件名即可
```
// 下载插件
npm install --save-dev babel-plugin-add-module-exports
npm install --save-dev babel-plugin-transform-es2015-modules-umd

// gulpfile babel 任务中指定插件
gulp.task('babel', function() {
    return gulp.src(['src/test.js'])
        .pipe($.babel({
            presets: ['es2015'],
            plugins: [
                "add-module-exports", "transform-es2015-modules-umd"
            ]
        }))
        .pipe(gulp.dest('bin/'));
});
```
经过编译后，最终生成的代码如下：
```
(function (global, factory) {
  if (typeof define === "function" && define.amd) {
    define(["module", "exports"], factory);
  } else if (typeof exports !== "undefined") {
    factory(module, exports);
  } else {
    var mod = {
      exports: {}
    };
    factory(mod, mod.exports);
    global.test = mod.exports;
  }
})(this, function (module, exports) {
  "use strict";

  Object.defineProperty(exports, "__esModule", {
    value: true
  });
  exports.default = 23;
  module.exports = exports["default"];
});
```



## 参考链接
[Misunderstanding ES6 Modules, Upgrading Babel, Tears, and a Solution](https://medium.com/@kentcdodds/misunderstanding-es6-modules-upgrading-babel-tears-and-a-solution-ad2d5ab93ce0)
