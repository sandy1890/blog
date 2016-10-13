---
title: 在多机部署时 Yii 的 assetManager 资源发布目录不一致问题
date: 2016-10-13 13:17:23
tags: 
    - PHP
categories:
    - 服务端
---

Yii 提供了 [assetManager](https://github.com/yiisoft/yii2/blob/master/framework/web/AssetManager.php) 来管理相对独立的资源内容，通过 assetManager 可以很方便地将相关功能的 js，css，图片等资源进行管理和二次发布。当我们的资源放置位置不是位于网络可访问目录中时，Yii 的 assetManager 会自动将这些资源自动发布到 @web/assets 目录中，并且随机生成一个资源文件夹名称。当我们的程序是单机部署时，没有问题。而当我们进行多机部署时，会发现在在每台机器上生成的资源文件夹名称不一致的情况。这将导致页面上部分资源文件无法加载，报 404 错误。

为了解决这个问题，我们先来看一下 Yii2 中关于资源文件夹目录名称生成的源码片断（文件位于　`web/AssetManager.php`）

```php
<?php
public function publish($path, $options = [])
{
    $path = Yii::getAlias($path);

    if (isset($this->_published[$path])) {
        return $this->_published[$path];
    }

    if (!is_string($path) || ($src = realpath($path)) === false) {
        throw new InvalidParamException("The file or directory to be published does not exist: $path");
    }

    if (is_file($src)) {
        return $this->_published[$path] = $this->publishFile($src);
    } else {
        return $this->_published[$path] = $this->publishDirectory($src, $options);
    }
}
```

在Yii内部，资源发布的时候调用的就是这个 [publish](https://github.com/yiisoft/yii2/blob/master/framework/web/AssetManager.php#L443)  函数，可以看到，这里面主要有两个相关函数 [publishFile](https://github.com/yiisoft/yii2/blob/master/framework/web/AssetManager.php#L468) 和 [publishDirectory](https://github.com/yiisoft/yii2/blob/master/framework/web/AssetManager.php#L513)

```php
<?php
protected function publishFile($src)
{
    $dir = $this->hash($src);
    $fileName = basename($src);
    $dstDir = $this->basePath . DIRECTORY_SEPARATOR . $dir;
    $dstFile = $dstDir . DIRECTORY_SEPARATOR . $fileName;
    if (!is_dir($dstDir)) {
        FileHelper::createDirectory($dstDir, $this->dirMode, true);
    }
    if ($this->linkAssets) {
        if (!is_file($dstFile)) {
            symlink($src, $dstFile);
        }
    } elseif (@filemtime($dstFile) < @filemtime($src)) {
        copy($src, $dstFile);
        if ($this->fileMode !== null) {
            @chmod($dstFile, $this->fileMode);
        }
    }
    return [$dstFile, $this->baseUrl . "/$dir/$fileName"];
}

protected function publishDirectory($src, $options)
{
    $dir = $this->hash($src);
    $dstDir = $this->basePath . DIRECTORY_SEPARATOR . $dir;
    if ($this->linkAssets) {
        if (!is_dir($dstDir)) {
            FileHelper::createDirectory(dirname($dstDir), $this->dirMode, true);
            symlink($src, $dstDir);
        }
    } elseif (!empty($options['forceCopy']) || ($this->forceCopy && !isset($options['forceCopy'])) || !is_dir($dstDir)) {
        $opts = array_merge(
            $options,
            [
                'dirMode' => $this->dirMode,
                'fileMode' => $this->fileMode,
            ]
        );
        if (!isset($opts['beforeCopy'])) {
            if ($this->beforeCopy !== null) {
                $opts['beforeCopy'] = $this->beforeCopy;
            } else {
                $opts['beforeCopy'] = function ($from, $to) {
                    return strncmp(basename($from), '.', 1) !== 0;
                };
            }
        }
        if (!isset($opts['afterCopy']) && $this->afterCopy !== null) {
            $opts['afterCopy'] = $this->afterCopy;
        }
        FileHelper::copyDirectory($src, $dstDir, $opts);
    }

    return [$dstDir, $this->baseUrl . '/' . $dir];
}
```

通过源码我们可以看到，这两个函数在生成随机目录名 `dir` 时实际上都调用了一个 [hash](https://github.com/yiisoft/yii2/blob/master/framework/web/AssetManager.php#L596)方法,让我们来看一下这个方法:
```php
<?php
protected function hash($path)
{
    if (is_callable($this->hashCallback)) {
        return call_user_func($this->hashCallback, $path);
    }
    $path = (is_file($path) ? dirname($path) : $path) . filemtime($path);
    return sprintf('%x', crc32($path . Yii::getVersion()));
}
```

到这里，应该能很明白的看到是什么原因导致了在多机器上部署会导致文件名不一致了。核心原因就在于 `filemtime($path)` 这个部分。filemtime() 函数的作用是返回文件内容上次的修改时间。多机器部署的时候，我们通常不能保证同一个文件在每台机器上的这个时间一致。所以导致最终计算出来的名称不一致。

现在让我们来看一下，如何解决这个问题，在上面的代码中，我们看到有一个 `hashCallBack` 属性，这个属性值是一个可执行的自定义资源目录生成函数。

解决方法一:　配置文件中全局设置 assetManager 组件

```php
<?php
components => [
    'assetManager' => [
        'hashCallback' => function ($path) {
            $path = (is_file($path) ? dirname($path) : $path);
            return sprintf('%x', crc32($path . Yii::getVersion()));
        },
    ],
]
```

解决方法一:　局部动态设置

```php
<?php
Yii::$app->getAssetManager()->hashCallback = function ($path) {
    return 'datatable';
}
```
