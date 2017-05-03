---
title: Node开发技术栈
date: 2016-05-20 20:00:56
tags: 
  - nodejs
  - npm
  - yarn
categories:
  - 前端
---

### package

* `cross-env` 跨平台设置NODE_ENV

package.json文件中命令设置环境变量时
windows 环境下报错: 'NODE_ENV' 不是内部或外部命令，也不是可运行的程序或批处理文件。

```bash
"scripts": {
    "start": " NODE_ENV=development webpack-dev-server --host 0.0.0.0 --devtool eval --progress --color --profile",
    "deploy": "NODE_ENV=production webpack -p --progress"
}
```

* `rimraf` nodejs版 rm -rf命令
  尤其在window下，当删除node_modules文件夹时，会报目录层次太深删除报错的问题
