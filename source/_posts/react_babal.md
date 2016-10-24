---
title: Babel CLI 安装及使用
date: 2016-10-20 10:30:56
tags: 
  - react
  - jsx
  - babel6
categories:
  - 前端
---
## 背景
本地项目部分页面使用 react ,通常就是一两个jsx文件需要编译。所以不打算使用 webpack 等打包工具。只需要最简单的能通过命令行编译 ES6 语法编写的 jsx 源文件为普通js文件

## Babel CLI 的安装

在官方网站安装说明部分，建议安装 babel CLI 使用本地方式，而非全局方式。一是因为本地开发中，各个项目依赖的 Babel 版本不尽相同，另外一方面是让项目更加独立，不依赖于具体的机器环境

There are two primary reasons for this.
* Different projects on the same machine can depend on different versions of Babel allowing you to update one at a time.
* It means you do not have an implicit dependency on the environment you are working in. Making your project far more portable and easier to setup.

在项目根目录中运行
```bash
$ npm install --save-dev babel-cli babel-preset-latest
```

因为全局安装运行 Babel 弊大于利，如果你之前已经全局安装过 Babel,可以使用如下命令卸载全局的 Babel
```
$ npm uninstall --global babel-cli
```

## 使用 ES6 开发 React 需要安装的其它模块

* babel-preset-es2015 （ES2015(ES6) 语法转换支持）
* babel-preset-react （JSX 语法转换支持）

在项目根目录中运行
```bash
$ npm install --save-dev babel-preset-es2015
$ npm install --save-dev babel-preset-react
```

## 监听并实时编译指定文件　

准备工具完成后，我们可以开始编写代码了。在代码编写过程中，通过如下命令实时监控代码变动并编译生成转换文件
```bash
babel 源文件名 --watch --presets es2015,react --out-file 转换后的文件名
```

我们也可以使用如下命令，将 src 目录下的所有文件编译到 lib 目录下
```bash
babel src --watch --presets es2015,react  --out-dir lib
```

## 参考链接
* [走进Babel 6.0 全新特性解析](http://www.csdn.net/article/2015-11-17/2826233)
* [Babel5 升级到 Babel6 总结](http://blog.csdn.net/windfromcn/article/details/51307153)
* [bablejs 官网](http://babeljs.io/)
* [ECMAScript 6 入门](http://es6.ruanyifeng.com/)