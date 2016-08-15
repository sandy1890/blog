---
title: Storm基础
date: 2016-04-19 13:37:56
categories:
    - 服务端
---

## Storm

### 术语

1. spout 龙卷，读取原始数据为bolt提供数据
2. bolt 雷电，从spout或其它bolt接收数据，并处理数据，处理结果可作为其它bolt的数据源或最终结果
3. nimbus 雨云，主节点的守护进程，负责为工作节点分发任务
4. topology 拓扑结构，Storm的一个任务单元
5. define field(s) 定义域，由spout或bolt提供，被bolt接收

### 安装

#### 设置zookeeper(单机服务)

* 下载zookeeper [官方地址](http://hadoop.apache.org/zookeeper/releases.html) [国内镜像](http://mirror.bit.edu.cn/apache/zookeeper)
* 解压下载文件，设置配置文件 conf/zoo.cfg

    ```ini
    tickTime=200
    dataDir=/var/zookeeper
    clientPort=2181
    ```

* 参数说明
    - **tickTime**
    Zookeeper基本时间单位.此单位用作心跳检测间隔以及session过期时间,session最小过期时间为此基本时间的2倍

    - **dataDir**
    若未在其它地方指定，则此目录作为内存数据快照和数据更新日志存储目录

    - **clientPort**
    指定服务监听端口,供客户端连接

* 启动zookper

     ```sh
        bin/zkServer.sh start
     ```

#### 在运行Nimbus和worker的机器上安装所需环境依赖
    * Java 6
    * Python 2.6.6

#### 在运行Nimbus和worker的机器上下载安装strom
    [下载页](http://www.apache.org/dyn/closer.lua/storm/apache-storm-0.9.6/apache-storm-0.9.6.tar.gz)

    下载后直接解压到安装目录即可

#### 配置strom安装目录下的 **conf/storm.yaml** 文件

```
storm.zookeeper.servers:
    - 127.0.0.1
storm.zookeeper.port: 2181
storm.local.dir: "/tmp/storm"
nimbus.host: "127.0.0.1"
supervisor.slots.ports:
    - 6700
```

**对于工作结点机器，storm.zookeeper.servers 和 nimbus.host 需要配置为相应服务器的访问地址**

#### 启动守护进程

* **Nimbus 启动主节点的守护进程**
    ```
    bin/storm nimbus
    ```

* **Supervisor 启动工作节点**
　
    ```
    bin/storm supervisor
    ```

* **UI 网页界面,可访问主节点主机8080端口 http://{nimbus host}:8080**
    ```
    bin/storm ui
    ```


#### 注册topology
storm jar target/storm-shopapi-1.0.0.jar com.mi.be.pls.strom.Topology

mvn exec:java -Dexec.mainClass="com.mi.be.pls.strom.Topology"


#### 文档
根目录执行

````
mvn javadoc:javadoc
mvn javadoc:aggregate -DreportOutputDirectory=./docs/ -DdestDir=javadocs
````

```
cd docs
jekyll serve -w
```
