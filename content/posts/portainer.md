---
title: "Portainer 的使用"
date: 2022-10-18T22:38:15+08:00
draft: false
---

### 运行面板

```shell
docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --name prtainer-test docker.io/portainer/portainer
```


### 安装 Agent

在Portainer的架构中，管理节点是Portainer Server，被管理节点通过部署Agent来与Server通信。

按照网络环境的不同，Portainer将Agent分为两种： `Portainer Agent` 和 `Edge Agent`。

1. `Portainer Agent`：当被管理服务器位于公网时，有公网IP，Server可以主动与其连接。此时，在被管理服务器上部署Portainer Agent来实现与Server之间的通信。

   > https://docs.portainer.io/v/be-2.12/start/install/agent/docker/linux

2. `Edge Agent`： 当被管理服务器位于内网时，Server无法主动与其连接。此时，需要在被管理服务器上部署Edge Agent，Edge Agent会周期性的从Server中获取需要执行的任务，从而实现与Server之间的通信。

   > https://docs.portainer.io/v/be-2.12/start/install/agent/edge

说白了就是一个主动一个被动

#### 安装 agent

```shell
docker run -d -p 9001:9001 --name portainer_agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent:2.12.1
```



### 配置 node

在面板中增加 endpoint 之后就可以使用了

