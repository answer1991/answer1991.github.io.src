````bash

$ docker run --net=host --privileged --name=calico-node -d --restart=always \
  -e IP_AUTODETECTION_METHOD=first-found \
  -e IP6_AUTODETECTION_METHOD=first-found \
  -e CALICO_LIBNETWORK_CREATE_PROFILES=true \
  -e CALICO_LIBNETWORK_IFPREFIX=cali \
  -e NODENAME=$(hostname) \
  -e CALICO_IPV4POOL_CIDR=192.168.0.0/16 \
  -e CALICO_LIBNETWORK_ENABLED=true \
  -e CALICO_LIBNETWORK_LABEL_ENDPOINTS=false \
  -e ETCD_ENDPOINTS=http://10.244.42.35:2379 \
  -e CALICO_NETWORKING_BACKEND=bird \
  -v /var/log/calico:/var/log/calico \
  -v /var/run/calico:/var/run/calico \
  -v /lib/modules:/lib/modules \
  -v /run/docker/plugins:/run/docker/plugins \
  -v /var/run/docker.sock:/var/run/docker.sock \
  quay.io/calico/node:v1.3.0
````