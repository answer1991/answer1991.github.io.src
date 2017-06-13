+++
date = "2017-06-13T19:21:00"
draft = false
tags = ["flannel", "docker", "network"]
title = "Use Flannel Network"
math = true
summary = """
Create a beautifully simple personal or academic website in under 10 minutes. 
"""

## [header]
## image = "headers/getting-started.png"
## caption = "Image credit: [**Academic**](https://github.com/gcushen/hugo-academic/)"

+++

### 创建etcd

````
$ docker run -dit --name etcd-1 --net host \
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
	-v $(pwd)/etcd:/infra1.etcd \
	acs-reg.alipay.com/k8s/etcd-amd64:3.0.10 \
	etcd --quota-backend-bytes=8589934592
````

### Create Network

````
$ curl -sSL http://localhost:2379/v2/keys/coreos.com/network/config -XPUT \
      -d value='{ "Network": "192.168.0.0/16", "Backend": {"Type": "udp"}}'
````

### Create flannel

````
$ docker run -dit --name=flanel_1 \
	--restart=always \
	--net=host \
	--privileged \
	-v /dev/net:/dev/net \
	-v /run/flannel:/run/flannel \
	acs-reg.alipay.com/k8s/flannel:v0.6.2-amd64 \
	/opt/bin/flanneld \
	--etcd-endpoints=http://127.0.0.1:2379 \
	--ip-masq=true \
	--iface=$(ip -o -4 addr list $(ip -o -4 route show to default | awk '{print $5}' | head -1) | awk '{print $4}' | cut -d/ -f1 | head -1)
	

````


### Restart Docker

````
$ DOCKER_CONF=$(sudo systemctl cat docker | head -1 | awk '{print $2}');source /run/flannel/subnet.env


if [[ -z $(grep -- "--mtu=" $DOCKER_CONF) ]]; then
    sed -e "s@$(grep "$SEARCH_FOR" $DOCKER_CONF)@$(grep "$SEARCH_FOR" $DOCKER_CONF) --mtu=${FLANNEL_MTU}@g" -i $DOCKER_CONF
  fi
  if [[ -z $(grep -- "--bip=" $DOCKER_CONF) ]]; then
    sed -e "s@$(grep "$SEARCH_FOR" $DOCKER_CONF)@$(grep "$SEARCH_FOR" $DOCKER_CONF) --bip=${FLANNEL_SUBNET}@g" -i $DOCKER_CONF
  fi
  
$  sed -e "s@$(grep -o -- "--mtu=[[:graph:]]*" $DOCKER_CONF)@--mtu=${FLANNEL_MTU}@g;s@$(grep -o -- "--bip=[[:graph:]]*" $DOCKER_CONF)@--bip=${FLANNEL_SUBNET}@g" -i $DOCKER_CONF

````


### utils

获取本地IP：

````
$ ip -o -4 addr list $(ip -o -4 route show to default | awk '{print $5}' | head -1) | awk '{print $4}' | cut -d/ -f1 | head -1
````