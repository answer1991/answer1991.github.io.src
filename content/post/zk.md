+++
date = "2017-06-13T19:21:00"
draft = false
tags = ["zk", "discovery", "docker"]
title = "分布式集群基础组件-Zookeeper"
math = true
summary = """
Zookeeper是Apache提供的分布式配置中心服务。本文介绍如何用Docker容器方式启动Zookeeper。
"""

##[header]
## image = "post/zk_logo.png" 
## caption = "Image credit: [**Academic**](https://github.com/gcushen/hugo-academic/)"

+++


### 单节点测试服务

````bash
$ docker run -dit \
    --name zk-single \
    --net host \
    --restart always \
    -v zk-data:/data \
    -v zk-log:/datalog \
    zookeeper:3.4.10
````

注意： 这里使用了``host``网络模式，这种基础组件服务我也建议使用``host``模式。
同时，服务会占用``2181``, ``2888``以及 ``3888``端口。


### 三节点生产集群

这里介绍没有使用``docker-compose``方式起服务。

假设我们要将服务器在 ``10.10.10.1``, ``10.10.10.2``, ``10.10.10.3`` 三台机器上。


##### 10.10.10.1节点

````bash
$ docker run -dit \
    --name zk-1 \
    --net host \ 
    --restart always \
    -v zk-data:/data \
    -v zk-log:/datalog \
    -e ZOO_MY_ID=1 \
    -e "ZOO_SERVERS=server.1=10.10.10.1:2888:3888 server.2=10.10.10.2:2888:3888 server.3=10.10.10.3:2888:3888" \
    zookeeper:3.4.10
````

##### 10.10.10.2节点

````bash
$ docker run -dit \
    --name zk-2 \
    --net host \
    --restart always \
    -v zk-data:/data \
    -v zk-log:/datalog \
    -e ZOO_MY_ID=2 \
    -e "ZOO_SERVERS=server.1=10.10.10.1:2888:3888 server.2=10.10.10.2:2888:3888 server.3=10.10.10.3:2888:3888" \
    zookeeper:3.4.10
````

##### 10.10.10.3节点

````bash
$ docker run -dit 
    --name zk-3 \
    --net host \
    --restart always \
    -v zk-data:/data \
    -v zk-log:/datalog \
    -e ZOO_MY_ID=3 \
    -e "ZOO_SERVERS=server.1=10.10.10.1:2888:3888 server.2=10.10.10.2:2888:3888 server.3=10.10.10.3:2888:3888" \
    zookeeper:3.4.10
````

