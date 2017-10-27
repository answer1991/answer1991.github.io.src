+++
date = "2017-06-23T19:21:00"
draft = false
tags = ["consul", "discovery", "docker"]
title = "分布式集群基础组件-Consul"
math = true
summary = """
Consul和ZooKeeper、etcd一样提供了分布式、强一致性的KV存储服务。相比于ZooKeeper、etcd，Consul将服务发现、健康监测、多机房等提到了自身服务层面，不需要做额外开发。
在使用多了之后，我更加喜欢使用Consul作为分布式KV存储和服务发现。
"""

##[header]
## image = "post/zk_logo.png" 
## caption = "Image credit: [**Academic**](https://github.com/gcushen/hugo-academic/)"

+++

### Server 

##### Bootstrap Node

````bash
$ docker run --net=host -dit --name=consul-server-bootstrap \
    -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' \
    -v consul-config:/consul/config \
    -v consul-data:/consul/data \
    consul:0.8.4 \
    agent -server \
    -client=0.0.0.0 \
    -bind $(hostname -i) \
    -bootstrap \
    -ui \
    -domain=consul \
    -datacenter=primary
````

##### Other Node

````bash
$ docker run --net=host -dit --name=consul-server \
    -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' \
    -v consul-config:/consul/config \
    -v consul-data:/consul/data \
    consul:0.8.4 \
    agent -server \
    -client=0.0.0.0 \
    -bind=$(hostname -i) \
    -retry-join=bootstrapNode \
    -ui \
    -domain=consul \
    -datacenter=primary
````

### Client

````bash
$ docker run --net=host -dit --name=consul-client \
    -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' \
    -v consul-config:/consul/config \
    -v consul-data:/consul/data \
    consul:0.8.4 \
    agent \
    -client=0.0.0.0 \
    -bind=$(hostname -i) \
    -retry-join=ServerNodes \
    -domain=consul
````

### DNSMASQ

````bash
$ sudo docker run -dit --net=host --name=dnsmasq \
    --cap-add=NET_ADMIN \
    andyshinn/dnsmasq:2.75 \
    -S /consul/127.0.0.1#8600
````