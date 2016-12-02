---
title: Go学习01 -- 环境搭建
date: 2016-12-02 15:40:56
categories:
    - 服务端
    - Go
---

### 安装下载

[官网](https://golang.org/dl/) 下载自己机器系统对应的二进制安装包，解压到相应目录.

解压目录说明：
* Linux 和 Mac OS　系统通常解压到 `/usr/local/go`
* Windows 系统通常解压到 `c:\Go` 目录

本次安装过程实际在 Ubuntu 机器上执行的命令如下:

```bash
#tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
sudo tar -C /usr/local -xzf go1.7.4.linux-amd64.tar.gz
```

### 环境变量设置 

* 在系统 PATH 环境变量里添加　 /usr/local/go/bin 
* GOROOT变量指向go安装目录，默认通常为 `/usr/local/go` 或 `c:\Go`,需要根据实际安装情况自行调整
* GOPATH 是用来设置包加载路径的重要变量，当在命令行运行 go build, go get 等命令时会需要该变量。这个变量通常指向你的工作空间（go项目代码存放位置）目录，这里假设你的工作空间为 $HOME/work (实际开发过程中为了方便管理项目，通常在项目自定义脚本中设置 GOPATH 和编译打包，省去每新增一个项目都需要修改 GOPATH 变量的麻烦，参见下文)

本将安装过程 Ubuntu 机器上新建 `/etc/profile.d/go.sh` 文件内容如下：

```bash
export GOROOT=/usr/local/go
export GOPATH=$HOME/work
export PATH=$PATH:$GOROOT/bin
```

创建完go.sh文件后，注销用户重新登录或是运行 

```bash
source /etc/profile
```

注：关于环境变量相关的设置，需要了解各平台相关知识，这里不细述，自行百度或谷歌

### Go 项目目录结构

一个 Go 程序项目一般包含三个目录

* src 目录存放go程序源文件,程序源文件按包组织（通常每个目录都对应一个包）
* pkg 目录包含包对象
* bin 目录包含可执行命令

一个工作空间目录结构看起来像这样:

```bash
bin/
    streak                         # 可执行命令
    todo                           # 可执行命令
pkg/
    linux_amd64/
        code.google.com/p/goauth2/
            oauth.a                # 包对象
        github.com/nf/todo/
            task.a                 # 包对象
src/
    code.google.com/p/goauth2/
        .hg/                       # mercurial 代码库元数据
        oauth/
            oauth.go               # 包源码
            oauth_test.go          # 测试源码
    github.com/nf/
        streak/
        .git/                      # git 代码库元数据
            oauth.go               # 命令源码
            streak.go              # 命令源码
        todo/
        .git/                      # git 代码库元数据
            task/
                task.go            # 包源码
            todo.go                # 命令源码
```

此工作空间包含三个代码库（goauth2、streak 和 todo），两个命令（streak 和 todo） 以及两个库（oauth 和 task）。

### 项目 GOPATH 设置脚本

前面介绍环境变量 GOPATH 的时候提到，为了避免每次新建一个项目都去修改系统环境变量的麻烦。我们可以使用一个脚本来设置特定项目的 GOPATH 变量。

install.sh 脚本:

```bash
#!/bin/bash
if [ ! -f install.sh ]; then
    echo '安装脚本必须在项目根目录中运行' 1>&2
    exit 1
fi
#获取当前项目目录
CURDIR=`pwd`
#获取系统设置的 GOPATH
OLDGOPATH="$GOPATH"
#将GOPATH设置为当前项目
export GOPATH="$CURDIR"

#以下为特定项目编译需要运行的命令
gofmt -w src
go build -o bin/hello src/hello.go
#以上为特定项目命令

#还原系统GOPATH变量值
export GOPATH="$OLDGOPATH"
echo '编译完成'
```

Windows 下的 install.bat 脚本(未验证):

```bash
@echo off

setlocal

if exist install.bat goto ok
    echo 安装脚本 install.bat 必须在项目根目录中运行
goto end

: ok

set OLDGOPATH=%GOPATH%
set GOPATH=%~dp0

gofmt -w src
go build -o bin/hello src/hello.go

:end
echo 编译完成
```

### hello,world

通过之前的介绍，现在我们来写一个hello,world的例子

* 项目目录结构如下:

```bash
helloworld/
├── bin
│   └── hello
├── install.sh
├── pkg
└── src
    └── hello.go
```


* 编写 `src\hello.go` 文件

```go
package main

import "fmt"

func main() {
    fmt.Printf("你好，世界.\n")
}
```


* 编写 `install.sh` 文件

```bash
#!/bin/bash
if [ ! -f install.sh ]; then
    echo '安装脚本必须在项目根目录中运行' 1>&2
    exit 1
fi
#获取当前项目目录
CURDIR=`pwd`
#获取系统设置的 GOPATH
OLDGOPATH="$GOPATH"
#将GOPATH设置为当前项目
export GOPATH="$CURDIR"


#以下为特定项目编译需要运行的命令
gofmt -w src
go build -o bin/hello src/hello.go
#以上为特定项目命令


#还原系统GOPATH变量值
export GOPATH="$OLDGOPATH"
echo '编译完成'
```


* 编译
编写完上面的文件之后，让我们进到项目根目录运行 helloworld程序

```bash
cd helloworld
#编译
sh install.sh
#运行
./bin/hello
#此时你应该能在终端看到如下输出
你好，世界.
```


### 本地运行Go自带的学习资源

* 离线运行官方文档
```bash
godoc -http=:8080 -v
```

* 离线运行官方入门学习教程
```bash
go tool tour
```

参考资源：
[官方文档中文版](http://docscn.studygolang.com/doc/install)
[Go项目的目录结构](http://blog.studygolang.com/2012/12/go%E9%A1%B9%E7%9B%AE%E7%9A%84%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84/)