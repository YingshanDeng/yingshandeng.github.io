title: 如何在 Github 上开源一个 JS 类库
tags:
---

<!-- more -->

## 创建仓库
```
```

## 封装成 UMD

## TEST
mocha: https://mochajs.org/

https://istanbul.js.org/docs/tutorials/mocha/

```
```

## test coverage

nyc https://istanbul.js.org/docs/tutorials/mocha/

https://github.com/istanbuljs/nyc#require-additional-modules

## JavaScript Standard Style
[](https://standardjs.com/readme-zhcn.html)
[](https://standardjs.com/rules-zhcn.html#javascript-standard-style)


遇到如下问题:

```
/Users/YingshanDeng/Documents/GitHub/EmojiCharString/test/test.js:5:1: 'describe' is not defined.
/Users/YingshanDeng/Documents/GitHub/EmojiCharString/test/test.js:6:3: 'it' is not defined.
```
[JavaScript Standard Style does not recognize Mocha](https://stackoverflow.com/questions/30018271/javascript-standard-style-does-not-recognize-mocha)

在 package.json 中添加：
```
"standard": {
    "env": [ "mocha" ]
  },
```
或者
```
"standard": {
    "global": [
      "describe",
      "it"
    ]
  },
```

## 其他

添加 badge 到 readme

1、npm version : https://badge.fury.io/for/js

## Travis CI


## Publishing npm packages
> 参考链接: [Publishing npm packages](https://docs.npmjs.com/getting-started/publishing-npm-packages)

### Creating a user
- 注册 npm registry 账号，两种方式：
	- 前往 [npm 官网](https://www.npmjs.com/) 注册
	- 使用 `npm adduser` 命令注册
- 执行 `npm login` 命令，输入 username, password, email 信息登录

### Publishing the package
- 修改项目中 `package.json` 文件中的 `version` 字段为发布版本号（版本号格式：`major.minor.patch`）
	> 关于版本号，参考: [论版本号的正确打开方式](http://taobaofed.org/blog/2016/08/04/instructions-of-semver/)
- 除了 `.gitignore` 或者 `.npmignore` 中忽略的文件外，其它文件都会发布到 npm
- 执行 `npm publish` 进行发布

### Updating the package
- 更新 `package.json` 文件中的 `version`，两种方式：
	- 手动修改版本号
	- 调用 `npm version <update_type>`(update_type: major / minor / patch) 对主版本号，次版本号和修订号某一项进行递增一
- 执行 `npm publish` 进行更新


## 参考链接
[]()
[]()