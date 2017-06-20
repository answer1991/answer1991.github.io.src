+++
date = "2017-06-13T19:21:00"
draft = false
tags = ["etcd", "discovery", "docker"]
title = "分布式集群基础组件-etcd"
math = true
summary = """
etcd使用广泛的的分布式配置中心服务，最常见的被用于k8s。本文介绍如何用Docker容器方式启动etcd。
"""

##[header]
## image = "post/zk_logo.png" 
## caption = "Image credit: [**Academic**](https://github.com/gcushen/hugo-academic/)"

+++


### 单节点测试服务

````bash
$ docker run -dit --name etcd-single --net host \
    --restart=always \
    -e ETCD_NAME=infra1 \
    -e ETCD_INITIAL_ADVERTISE_PEER_URLS=http://$(hostname -i):2380 \
    -e ETCD_LISTEN_PEER_URLS=http://$(hostname -i):2380 \
    -e ETCD_LISTEN_CLIENT_URLS=http://$(hostname -i):2379,http://127.0.0.1:2379 \
    -e ETCD_ADVERTISE_CLIENT_URLS=http://$(hostname -i):2379 \
    -e ETCD_INITIAL_CLUSTER_TOKEN=hello_world \
    -e ETCD_INITIAL_CLUSTER=infra1=http://$(hostname -i):2380 \
    -e ETCD_INITIAL_CLUSTER_STATE=new \
    -e ETCD_AUTO_COMPACTION_RETENTION=1 \
    -e ETCD_SNAPSHOT_COUNT=10000 \
    -e ETCD_MAX_SNAPSHOTS=5 \
    -e ETCD_MAX_WALS=5 \
    -e ETCD_HEARTBEAT_INTERVAL=1000 \
    -e ETCD_ELECTION_TIMEOUT=10000 \
    -v etcd-infra1:/infra1.etcd \
    quay.io/coreos/etcd:v3.2.0 \
    etcd --quota-backend-bytes=8589934592
````

注意： 这里使用了``host``网络模式，这种基础组件服务我也建议使用``host``模式。
同时，服务会占用``2379``, ``2380``端口。


### 三节点生产集群

这里介绍没有使用``docker-compose``方式起服务。

假设我们要将服务器在 ``10.10.10.1``, ``10.10.10.2``, ``10.10.10.3`` 三台机器上。

那么``ETCD_INITIAL_CLUSTER``的值为``infra1=http://10.10.10.1:2380,infra2=http://10.10.10.2:2380,infra3=http://10.10.10.3:2380``

##### 10.10.10.1节点

````bash
$ docker run -dit --name etcd-infra1 --net host \
    --restart=always \
    -e ETCD_NAME=infra1 \
    -e ETCD_INITIAL_ADVERTISE_PEER_URLS=http://$(hostname -i):2380 \
    -e ETCD_LISTEN_PEER_URLS=http://$(hostname -i):2380 \
    -e ETCD_LISTEN_CLIENT_URLS=http://$(hostname -i):2379,http://127.0.0.1:2379 \
    -e ETCD_ADVERTISE_CLIENT_URLS=http://$(hostname -i):2379 \
    -e ETCD_INITIAL_CLUSTER_TOKEN=hello_world \
    -e ETCD_INITIAL_CLUSTER=infra1=http://10.10.10.1:2380,infra2=http://10.10.10.2:2380,infra3=http://10.10.10.3:2380 \
    -e ETCD_INITIAL_CLUSTER_STATE=new \
    -e ETCD_AUTO_COMPACTION_RETENTION=1 \
    -e ETCD_SNAPSHOT_COUNT=10000 \
    -e ETCD_MAX_SNAPSHOTS=5 \
    -e ETCD_MAX_WALS=5 \
    -e ETCD_HEARTBEAT_INTERVAL=1000 \
    -e ETCD_ELECTION_TIMEOUT=10000 \
    -v etcd-infra1:/infra1.etcd \
    quay.io/coreos/etcd:v3.2.0 \
    etcd --quota-backend-bytes=8589934592
````

##### 10.10.10.2节点

````bash
$ docker run -dit --name etcd-infra2 --net host \
    --restart=always \
    -e ETCD_NAME=infra2 \
    -e ETCD_INITIAL_ADVERTISE_PEER_URLS=http://$(hostname -i):2380 \
    -e ETCD_LISTEN_PEER_URLS=http://$(hostname -i):2380 \
    -e ETCD_LISTEN_CLIENT_URLS=http://$(hostname -i):2379,http://127.0.0.1:2379 \
    -e ETCD_ADVERTISE_CLIENT_URLS=http://$(hostname -i):2379 \
    -e ETCD_INITIAL_CLUSTER_TOKEN=hello_world \
    -e ETCD_INITIAL_CLUSTER=infra1=http://10.10.10.1:2380,infra2=http://10.10.10.2:2380,infra3=http://10.10.10.3:2380 \
    -e ETCD_INITIAL_CLUSTER_STATE=new \
    -e ETCD_AUTO_COMPACTION_RETENTION=1 \
    -e ETCD_SNAPSHOT_COUNT=10000 \
    -e ETCD_MAX_SNAPSHOTS=5 \
    -e ETCD_MAX_WALS=5 \
    -e ETCD_HEARTBEAT_INTERVAL=1000 \
    -e ETCD_ELECTION_TIMEOUT=10000 \
    -v etcd-infra2:/infra2.etcd \
    quay.io/coreos/etcd:v3.2.0 \
    etcd --quota-backend-bytes=8589934592
````

##### 10.10.10.3节点

````bash
$ docker run -dit --name etcd-infra3 --net host \
    --restart=always \
    -e ETCD_NAME=infra3 \
    -e ETCD_INITIAL_ADVERTISE_PEER_URLS=http://$(hostname -i):2380 \
    -e ETCD_LISTEN_PEER_URLS=http://$(hostname -i):2380 \
    -e ETCD_LISTEN_CLIENT_URLS=http://$(hostname -i):2379,http://127.0.0.1:2379 \
    -e ETCD_ADVERTISE_CLIENT_URLS=http://$(hostname -i):2379 \
    -e ETCD_INITIAL_CLUSTER_TOKEN=hello_world \
    -e ETCD_INITIAL_CLUSTER=infra1=http://10.10.10.1:2380,infra2=http://10.10.10.2:2380,infra3=http://10.10.10.3:2380 \
    -e ETCD_INITIAL_CLUSTER_STATE=new \
    -e ETCD_AUTO_COMPACTION_RETENTION=1 \
    -e ETCD_SNAPSHOT_COUNT=10000 \
    -e ETCD_MAX_SNAPSHOTS=5 \
    -e ETCD_MAX_WALS=5 \
    -e ETCD_HEARTBEAT_INTERVAL=1000 \
    -e ETCD_ELECTION_TIMEOUT=10000 \
    -v etcd-infra3:/infra3.etcd \
    quay.io/coreos/etcd:v3.2.0 \
    etcd --quota-backend-bytes=8589934592
````
