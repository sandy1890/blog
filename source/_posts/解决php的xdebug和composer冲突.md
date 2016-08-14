---
title: 解决php的xdebug和composer冲突
date: 2016-05-13 11:52:10
tags: php
---
## [composer](http://www.phpcomposer.com)
Composer 是 PHP 的一个依赖管理工具。它允许你申明项目所依赖的代码库，它会在你的项目中为你安装他们

## [xdebug](https://xdebug.org)
Xdebug是一个开放源代码的PHP程序调试器(即一个Debug工具)，可以用来跟踪，调试和分析PHP程序的运行状况

## 问题
如果php安装了xdebug扩展，当使用composer的时候，会有如下警告提示。提示信息里的连接给出了[官方解决方案](https://getcomposer.org/xdebug)　（这个地址竟然也被天朝墙，还让不让干活了）

引起的主要问题就是当你使用composer的时候，下载速度会被得奇慢无比

> You are running composer with xdebug enabled. This has a major impact on runtime performance. See https://getcomposer.org/xdebug

官方的链接给出了以下几个解决方案

* 禁用xdebug，在*nix系统下，再利用别名来使得命令行直接调用php可加载xdebug。通过以下两步实现
    * php.ini配置里禁用xdebug

        ```bash
        ;zend_extension = "/path/to/my/xdebug.so"
        ```
    * *nix系统里定义命令别名
        ```bash
        # 直接调用php命令时加载xdebug
        alias php='php -dzend_extension=xdebug.so'
        # PHPUnit需要使用xdebug.那么定义一个别名来处理
        alias phpunit='php $(which phpunit)'
        ```

* 新建一个禁用了xdebug的xdebug-disabled-php.ini文件，还是使用别名，将composer命令重新定义
   ```bash
   # Without php.ini
    alias comp='php -n /path/to/composer.phar'
    # Or with an xdebug-disabled php.ini
    alias comp='php -c /path/to/xdebug-disabled-php.ini /path/to/composer.phar'
   ```

...官方给出了另外的复杂方案，太麻烦，就不贴了，有兴趣到[这里](https://getcomposer.org/xdebug)自行查看

## 我的解决方案

针对Linux平台,因为我自己开发机器用的是php-fpm启动脚本

主要诉求是希望我机器上开机默认启动php的时候，能加载xdebug扩展，命令行下运行composer调用php命令的时候，不加载。解决思路是，在php.ini文件中禁用xdebug扩展，然后在php-fpm启动脚本中增加参数，加载xdeubg

具体操作就是直接修改启动脚本`/etc/init.d/php-fpm`下的参数,增加`-dzend_extension=xdebug.so`(注意，这里你可能需要传入xdebug.so文件的绝对路径)

```bash
php-fpm -dzend_extension=xdebug.so
```
