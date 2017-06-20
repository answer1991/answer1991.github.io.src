+++
date = "2017-06-13T19:21:00"
draft = false
tags = ["flannel", "docker", "network"]
title = "使用flannel创建跨节点的docker容器网络"
math = true
summary = """
在多节点的docker集群环境下，docker自带的网络不能满足跨节点的容器互通问题。本文介绍使用``flannel``来搭建跨节点的虚拟docker容器网络。
"""

## [header]
## image = "headers/getting-started.png"
## caption = "Image credit: [**Academic**](https://github.com/gcushen/hugo-academic/)"

+++

## 背景

在多节点的docker集群环境下，docker自带的网络不能满足跨节点的容器互通问题。当然这个是有前提：容器不以``host``网络模式启动。
如果要实现互通，都以``host网络``模式启动会带来其它问题，如端口冲突。

``k8s``用``flannel``来解决上述问题。本文介绍单独使用``flannel``来搭建跨节点的虚拟docker容器网络。

### 1. 创建etcd

参考本站博客内的etcd博文。

### 2. 创建flannel Network Config

只需被创建一次，网段和backend定下里之后不能更改。

````bash
$ curl -sSL http://localhost:2379/v2/keys/coreos.com/network/config -XPUT \
      -d value='{ "Network": "192.168.0.0/16", "Backend": {"Type": "udp"}}'
````

### 3. 运行flanneld

假设``etcd``服务在``10.10.10.1``, ``10.10.10.2``, ``10.10.10.3``三台机器上，
那么``etcd-endpoints``的值为``http://10.10.10.1:2379,http://10.10.10.2:2379,http://10.10.10.3:2379``.

````bash
$ docker run -dit --name=flannel \
    --restart=always \
    --net=host \
    --privileged \
    -v /dev/net:/dev/net \
    -v /run/flannel:/run/flannel \
    quay.io/coreos/flannel:v0.7.1-amd64 \
    /opt/bin/flanneld \
    --etcd-endpoints=http://10.10.10.1:2379,http://10.10.10.2:2379,http://10.10.10.3:2379 \
    --ip-masq=true \
    --iface=$(ip -o -4 addr list $(ip -o -4 route show to default | awk '{print $5}' | head -1) | awk '{print $4}' | cut -d/ -f1 | head -1)
````


### 4. 修改docker daemon的bip和mtu参数

``bip``和``mtu``获取方式:

````bash
$ cat /run/flannel/subnet.env
 FLANNEL_NETWORK=192.168.0.0/16
 FLANNEL_SUBNET=192.168.66.1/24
 FLANNEL_MTU=1472
 FLANNEL_IPMASQ=true

````

其中 ``bip``为``FLANNEL_SUBNET``的值， ``mtu``的为``FLANNEL_MTU``的值。

将这两个参数加入到docker daemon启动配置中。以``systemctl``系统为例，可以执行一下脚本


````bash
 
DOCKER_CONF=$(sudo systemctl cat docker | head -1 | awk '{print $2}');

source /run/flannel/subnet.env

cp -f $DOCKER_CONF $DOCKER_CONF.backup

if [[ -z $(grep -- "--mtu=" $DOCKER_CONF) ]]; then
    sed -e "s@$(grep "$SEARCH_FOR" $DOCKER_CONF)@$(grep "$SEARCH_FOR" $DOCKER_CONF) --mtu=${FLANNEL_MTU}@g" -i $DOCKER_CONF
fi
if [[ -z $(grep -- "--bip=" $DOCKER_CONF) ]]; then
    sed -e "s@$(grep "$SEARCH_FOR" $DOCKER_CONF)@$(grep "$SEARCH_FOR" $DOCKER_CONF) --bip=${FLANNEL_SUBNET}@g" -i $DOCKER_CONF
fi
  
sed -e "s@$(grep -o -- "--mtu=[[:graph:]]*" $DOCKER_CONF)@--mtu=${FLANNEL_MTU}@g;s@$(grep -o -- "--bip=[[:graph:]]*" $DOCKER_CONF)@--bip=${FLANNEL_SUBNET}@g" -i $DOCKER_CONF

````

执行后的结果如下（注意``--bip``和``--mtu``）：

````bash
$ cat /etc/systemd/system/docker.service |grep ExecStart
ExecStart=/usr/bin/docker daemon -H fd:// --tlsverify --tlscacert /etc/docker/ca.pem --tlscert=/etc/docker/server.pem --tlskey=/etc/docker/server-key.pem --log-opt max-size=200m  --log-opt max-file=5 -H tcp://0.0.0.0:2376  --insecure-registry=acs-reg.sqa.alipay.net  --insecure-registry=alipay.docker.io  --registry-mirror=https://ant.mirror.aliyuncs.com --cluster-store= --storage-driver=overlay  --bip=192.168.66.1/24 --mtu=1472
````

### 5. 删除docker0，重启docker daemon

````bash
$ ip link set docker0 down
$ ip link del docker0

$ systemctl daemon-reload
$ systemctl restart docker
````

### 其它节点安装方式

``步骤2`` - ``步骤5``重复到其它节点上面。这样flannel网络建成了。

### 验证方式

选两个节点，分别创建容器，不用指定网络类型。如：

````bash
$ docker run -dit nginx
````

从一个节点ping另一个节点的容器的IP，可以ping通。

从一个节点的容器内去ping另一个节点的容器IP，可以ping通。

这样就实现了跨节点的docker容器虚拟网络。

### utils

获取本地IP：

````
$ ip -o -4 addr list $(ip -o -4 route show to default | awk '{print $5}' | head -1) | awk '{print $4}' | cut -d/ -f1 | head -1
````