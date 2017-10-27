+++
date = "2017-07-27T14:18:00"
draft = false
tags = ["docker", "bash", "ops"]
title = "Docker清理命令"
math = true
summary = """
Docker常用清理命令
"""

+++

引用自： https://www.calazan.com/docker-cleanup-commands/

#### Kill all running containers

````bash
docker stop $(docker ps -q)
````

#### Delete all stopped containers (including data-only containers)

````bash
docker rm $(docker ps -a -q)
````

#### Delete all 'untagged/dangling' (<none>) images

````bash
docker rmi $(docker images -q -f dangling=true)
````

#### Delete ALL images

````bash
docker rmi $(docker images -q)
````