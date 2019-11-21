---
layout: post
title: "本地Docker Hub配置"
subtitle: "Docker-Hub-Configration"
date: 2017-12-05 12:55:00
author: "Joshua"
#header-img: "img/RYB.jpg"
categories:
  - 技术
tags:
  - docker
---

> 网上配置贴非常多，但是因为系统版本和 docker 版本不同略有差异，本贴环境 ubuntu 16.04 docker-ce 17.0x

<!-- more -->

# 启动 regisrty 容器

```
docker run -d -p 5000:5000 -v /mnt/date/registry:/var/lib/registry registry
```

-d 后台运行

-p 指定端口

-v 把 registry 的镜像路径/var/lib/registry 映射到本机的/mnt/date/registry

```
curl 0.0.0.0:5000/v2/_catalog
```

返回目前 regisrty 存在的 image，不报错说明服务启动成功。

# 修改 docker https 配置

因为 docker 镜像的 push 和 pull 默认进过 https 传输，本地的 registry 并没有启动 https 的连接，所以要修改部分配置。

```
vim /lib/systemd/system/docker.service
```

打开 docker 配置文件，需要 root 权限

在 ExecStart 参数最后加上 --insecure-registry [registry_ip]:[registry_port],如下：

```
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock  --insecure-registry 192.168.23.113:5000
```

执行

```
systemctl daemon-reload
systemctl restart docker
```

重新加载 docker.service 文件的配置，并重启 docker 服务。

# 推送镜像到本地的 hub

首先要 tag 现有的 image

```
docker tag hyperledger/fabric-kafka 192.168.23.113:5000/fabric-kafka
```

然后推送 image 到本地镜像

```
docker push 192.168.23.113:5000/fabric-kafka
```

# 从本地 hub 拉取镜像

```
docker pull 192.168.23.113:5000/fabric-kafka
```
