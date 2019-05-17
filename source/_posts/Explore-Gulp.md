title: 探究 Gulp：基于流的自动化构建工具
date: 2018-02-27 22:21:40
tags: Gulp
categories: JS
---

![](http://cdn.objcer.com/gulpjs-cover.png)

Gulp 是三个自动化构建工具（Grunt, Gulp, Webpack）之一，其通过代码优于配置的策略，Gulp 让简单的任务简单，复杂的任务可管理，仅提供几个 API 就能将构建任务很好完成，而且有着丰富的插件生态，本文将探究 Gulp 的原理及如何创建插件。

<!-- more -->

## 流

### Node 中的流
Node 中的 Stream 一般分四种, 其中 Transform 也是 Duplex，只不过输出是由输入计算得到的，因此算作 Duplex 的特例。
- 输入流(stream.Readable)
- 输出流(stream.Writable)
- 读写流(stream.Duplex)
- 转换流(stream.Transform)

Stream 流有一个很基本的操作 `pipe` 管道。用于将一个流的输出口接到另一个流的输入口，起连接两个流的作用。
```
a.pipe(b).pipe(c).pipe(d)
```

Stream 都是 Node 事件对象 EventEmitter 的实例( All streams are instances of EventEmitter )，它们可以通过 `on` 添加事件监听。
```js
var fs = require("fs");
fs.createReadStream('src.txt')
  .pipe(fs.createWriteStream('dest.txt'))
  .on('finish', () => {
    console.log("Write complete.")
  })
```

`fs.createReadStream('src.txt')` 创建了 Readable Stream，`fs.createWriteStream()` 创建了 Writable Stream，然后通过管道方法 `pipe` 就可以完成数据从 Readable Stream 到 Writable Stream 的流动。Stream 是 EventEmitter 的实例，通过 `on` 方法为 Readable Stream 添加了一个 `finish` 事件，写入完成时触发该事件回调。

### Stream 和 Buffer
**两种数据类型：**
- buffer：数据缓冲对象，是一个类似数组结构的对象，可以通过指定开始写入的位置及写入的数据长度，往其中写入**二进制数据**
- stream：也就是我们上面提到的 Node 中的流，stream 对 buffer 对象的高级封装，其操作的底层还是 buffer 对象

**处理数据的两种模式：**
- Buffer 模式：就是取完整数据一次性操作
- Stream 模式：就是边读取数据边操作

例如打开一个 2G 的文件, 用 Buffer 模式就是先分配 2G 的内存, 把文件全部读出来, 然后开始操作内存；而用 Stream 模式的方法就是边读数据边处理。可见，Steam 模式无论在空间上还是时间上都优于 Buffer 模式。

## vinyl
[vinyl](https://github.com/gulpjs/vinyl) 是一种虚拟文件格式（Virtual file format），也是一种 JavaScript 对象，称之为：**Vinyl File Object**，如下：
```
var Vinyl = require('vinyl');

var jsFile = new Vinyl({
  cwd: '/',
  base: '/test/',
  path: '/test/file.js',
  contents: new Buffer('var x = 123')
});
```

我们知道 Gulp 是基于流的自动化构建工具，但是这个流并非 Node 中的 Stream，而是 **Vinyl File Object Stream**，但其中用到了 Node 中的 Stream。那 Gulp 为什么不直接使用 Node 中的 Stream 呢？看如下例子：
```js
gulp.task("copy", function(){
  return gulp.src("./stylesheets/src/**/*.css")
    .pipe(gulp.dest("./stylesheets/dest"));
});
```

这段代码将 `src` 目录下的 css 文件全部拷贝到 `dest` 目录，拷贝过程中保留了目录结构，文件名等等。 Node 中的 Stream 只传输 String 类型（Buffer经过 `utf8` 编码），Buffer 类型（二进制）和 Null（空），也即只关注了文件的内容；但是 Gulp 中不只关注文件内容，还用到这个文件相关的路径信息，而这是 Node 中的 Stream 无法支持的。留意上面 vinyl 对象代码，我们会发现包含了 `contents` 和 `path` 属性，所以 Gulp 中用到 Vinyl File Object，其使用的流就是 Vinyl File Object Stream。

### vinyl-fs
[vinyl-fs](https://github.com/gulpjs/vinyl-fs) 是 vinyl 文件对象的适配器(Vinyl adapter for the file system)。Gulp 中并没有直接使用 vinyl 文件对象，而是通过 vinyl-fs 提供的方法进行操作，在 [Gulp 源码](https://github.com/gulpjs/gulp/blob/master/index.js) 中发现：
```
var vfs = require('vinyl-fs');

Gulp.prototype.src = vfs.src;
Gulp.prototype.dest = vfs.dest;
```

### vinyl 内容类型
Vinyl File Object 的 `contents` 属性有三种类型：
- stream
- buffer(二进制)
- null

Gulp 插件中操作的虽然都是 Vinyl File Object，但是对 `contents` 类型可能会有不同的要求。在使用 Gulp 过程中可能会遇到这样的插件报错：
```
internal/streams/legacy.js:59
      throw er; // Unhandled stream error in pipe.
      ^
GulpUglifyError: Streaming not supported
    at createError (/Users/YingshanDeng/Documents/GitHub/SharedPen/node_modules/gulp-uglify/lib/create-error.js:6:14)
    at apply (/Users/YingshanDeng/Documents/GitHub/SharedPen/node_modules/lodash/_apply.js:16:25)
    at wrapper (/Users/YingshanDeng/Documents/GitHub/SharedPen/node_modules/lodash/_createCurry.js:41:12)
    at /Users/YingshanDeng/Documents/GitHub/SharedPen/node_modules/gulp-uglify/lib/minify.js:32:15
    ...
```

前面介绍了 Stream 和 Buffer 的区别，我们知道：Buffer 就是一次性获取完整的文件数据，而 Stream 是将文件切分成小块，一块一块获取；那么对于不同的插件可能有不同的要求。例如上面报错的就是 gulp-uglify 插件，提示不支持 Stream，需要 Buffer 类型的 Vinyl File Object。

`gulp.src(globs[, options])` 方法默认会返回 buffer 类型的 Vinyl File Object。
```
options.buffer
  Type: Boolean Default: true

  Setting this to false will return file.contents as a stream and not buffer files.
  This is useful when working with large files.
  Note: Plugins might not implement support for streams.
```

想要 Steam 类型可通过设置 `options.buffer` 为 `false`
```
gulp.src("*.js", {buffer: false})
```

### 类型转换
在编写插件过程通过会遇到 Node Stream 和 Vinyl File Object Stream 之间的转换，下面介绍几种情况：

- 将 Node Stream 转换成 Vinyl File Object Stream
  - 转换成 `contents` 类型是 stream 的 Vinyl File Object Stream
  使用 [vinyl-source-stream](https://www.npmjs.com/package/vinyl-source-stream) 插件
  - 转换成 `contents` 类型是 buffer 的 Vinyl File Object Stream
  使用 [vinyl-source-buffer](https://www.npmjs.com/package/vinyl-source-buffer) 插件
- 将 stream 类型的 Vinyl File Object Stream 转换成 buffer 类型的 Vinyl File Object Stream
  使用 [vinyl-buffer](https://www.npmjs.com/package/vinyl-buffer) 插件

下面通过一个经典的例子来介绍一下这些插件的使用：
```js
var browserify = require('browserify')
var source = require('vinyl-source-stream')
var buffer = require('vinyl-buffer')

gulp.task('bundle', () => {
  return browserify(bundle.entry, { standalone: bundle.standalone })
    .transform(babelify, {
      presets: ['es2015']
    })
    .bundle() // ①
    .pipe(source('filename.js')) // ②
    .pipe(gulp.dest('dist/'))
    .pipe(buffer()) // ③
    .pipe(uglify())
    .pipe(rename({
      extname: '.min.js'
    }))
    .pipe(gulp.dest('dist/'))
})
```

① [browserify](http://browserify.org/) 模块打包工具的 bundle 方法将打包好的文件生成 Node Stream 中的 Readable Stream
② vinyl-source-stream 插件将 Node Stream 转换成 Vinyl File Object Stream，这样才能在 Gulp 中使用（注意：需要传入一个文件名）
③ 将 stream 类型的 Vinyl File Object Stream 转换成 buffer 类型的 Vinyl File Object Stream，满足 gulp-uglify 插件的使用要求

## 编写 Gulp 插件
> [官方编写插件文档](https://www.gulpjs.com.cn/docs/writing-a-plugin/)

简单的理解，Gulp 插件就是通过接收 Vinyl File Object，然后进行插件逻辑处理，最后返回 Vinyl File Object。这个过程也就是对流进行转换(transform stream)，通常我们会使用到 [through2](https://github.com/rvagg/through2)。下面通过一个例子了解一下如何编写一个 Gulp 插件：

`index.js`
```
const through2 = require('through2');
const PLUGIN_NAME = 'gulp-demo'

module.exports = function () {
  return through2.obj(function(chunk, encoding, callback) {
    if (chunk.isNull()) {
      callback(null, chunk)
      return
    }
    if (chunk.isStream()) {
      this.emit('error', new Error(`${PLUGIN_NAME}: Streaming not supported`))
      callback()
      return
    }
    if (chunk.isBuffer()) {
      // 向文件内容追加 'demo'
      chunk.contents = new Buffer(chunk.contents.toString() + 'demo')
      this.push(chunk)
      cb()
    }
  })
}
```

`gulpfile.js`
```js
var gulp = require('gulp')
var fs = require('fs')
var demoPlugin = require('./index.js')
var source = require('vinyl-source-stream')
var buffer = require('vinyl-buffer')

gulp.task('demo', () => {
  return fs.createReadStream('./test/src/text.txt')
    .pipe(source('text.txt'))
    .pipe(buffer())
    .pipe(demoPlugin())
    .pipe(gulp.dest('./test/dest'))
})
// 或者
// gulp.task('demo', () => {
//   return gulp.src('./test/src/text.txt')
//     .pipe(demoPlugin())
//     .pipe(gulp.dest('./test/dest'))
// })
```

## Gulp 和 Grunt 的比较
Gulp 基于流； 而 Grunt 基于文件的机制，导致了任务之间没有信息传递。
Grunt 任务流程基本上打开文件、处理文件、保存文件、关闭文件，然后执行继续向后执行任务。每一个任务都需要做重复的打开、保存、关闭操作无疑影响效率。Gulp 的特点在于单入口模式，文件打开、保存、关闭均一次，都是在内存中获取数据，操作数据，这样肯定比 Gulp 快。

## 参考链接
[Gulp 中文网](https://www.gulpjs.com.cn/)
[探究 Gulp](https://segmentfault.com/a/1190000003770541)

