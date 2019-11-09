---
layout:     post
title:      "在 Swarm 集群环境部署 Fabric 区块链"
subtitle:   "Deploy fabric in swarm"
date:       2017-12-07 12:55:00
author:     "Joshua"
header-img: "img/fabric-in-swarm/bg.jpg"
categories:
  - 区块链技术
tags:
    - Hyperledger
    - 区块链
    - docker
    - swarm
    - fabric
---
## 前言
fabric 部署本来就是一件麻烦的事情，如果还涉及到多机环境部署那就更加的麻烦了。
因为 fabric 推荐部署在docker环境中，那很自然地想到通过集群部署工具来部署fabric。本文就通过例子来说明如何在 Swarm 集群中部署 fabric 区块链网络。

还有之前看到过张海宁的[k8s部署方案](https://github.com/hainingzhang/articles/blob/master/fabric_multi_nodes/FabricMultiNodev2-3.pdf)，供大家参考。

文中所有IP地址,如192.168.1.93，都需要根据你本地的情况输入。

本例子github地址：[fabric-in-swarm](https://github.com/qiushaoxi/fabric-in-swarm.git) , 欢迎star.
## 架构说明
![架构图](/img/fabric-in-swarm/architect.png)

docker swarm 会在各台机器之上 创建一个 overlay 虚拟网络，通过swarm启动的container可以跨主机互相访问。

docker service 是 swarm 编排的最小单位，可以配置运行的镜像、网络端口、环境变量等，一个service 可以启动多个container进行负载均衡。不过因为区块链中每个节点都有不同的配置，本例子中每个service只跑一个container。后续可以继续探讨启动多个container的场景。

docker stack 则是一组 service 的合集， 每个stack会默认创建一个 default 的 overlay 网络，其中还可以再配置其他网络。 stack 本意是技术栈，通过在 stack 中配置前后端、中间件等 service，可以实现一键部署整个技术栈。本例子的stack 中，配置了1个orderer、4个peer和1个cli一共6个service。后续可以根据需要加入kafka、couchdb等service。

swarm 集群会对外暴露一个IP，通过端口映射，可以访问到具体 service 的端口。
## 基础环境搭建
### Swarm 集群搭建
#### 安装 docker
如果还没有安装 docker ，执行：
```
curl -fsSL https://get.docker.com | sh
```
#### 在 leader 创建 Swarm 集群
选择一台机器作为 leader
```
docker swarm init --advertise-addr 192.168.1.93
```
这里的ip是整个swarm集群对外的ip，所有swarm集群里面的容器都通过这个ip对外提供服务。

执行返回：
```
To add a worker to this swarm, run the following command:
 
    docker swarm join /
    --token SWMTKN-1-16kit6dksvrqilgptjg5pvu0tvo5qfs8uczjq458lf9mul41hc-dzvgu0h3qngfgihz4fv0855bo /
    192.168.1.93:2377
 
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
#### 其他机器加入 Swarm 集群
根据第一步返回的命令加入集群：
```
docker swarm join /
--token SWMTKN-1-16kit6dksvrqilgptjg5pvu0tvo5qfs8uczjq458lf9mul41hc-dzvgu0h3qngfgihz4fv0855bo /
192.168.1.93:2377
```
所有其他机器执行命令后，在leader执行
```
docker node ls
```
如果能看到所有机器，说明操作成功，Swarm集群搭建完毕。

#### portainer
这里推荐一个开源的docker后台管理工具[portainer](https://portainer.io/)，也支持Swarm集群的管理。
```
docker service create \
--name portainer \
--publish 9000:9000 \
--replicas=1 \
--constraint 'node.role == manager' \
--mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock \
portainer/portainer \
-H unix:///var/run/docker.sock
```
执行这个命令就可以在Swarm中启动，然后访问 Swarm_IP:9000 就可以进行管理后台，初次需要设置密码。

### NFS服务器搭建
因为集群容器需要读取一些文件，所以需要搭一个文件服务器。
#### 安装nfs-kernel-server
可以在任意台机器安装，本例子就在 swarm manager的机器上安装的。
```
apt install nfs-common
apt install nfs-kernel-server
```
#### 修改配置文件
```
vim /etc/exports
```
修改内容如下：
```
/nfs *(rw,sync,no_root_squash)
```
各段表达的意思如下，根据实际进行修改
```
/nfs   ：共享的目录
*       ：指定哪些用户可以访问
            *  所有可以ping同该主机的用户
            192.168.1.*  指定网段，在该网段中的用户可以挂载
            192.168.1.12 只有该用户能挂载
(ro,sync,no_root_squash)：  权限
        ro : 只读
        rw : 读写
        sync :  同步
        no_root_squash: 不降低root用户的权限
    其他选项man 5 exports 查看
```
#### 创建目录并重启nfs服务
```
mkdir /nfs
service nfs-kernel-server restart
```
到此，nfs的服务就搭建好了。
#### 连接挂载nfs服务器
执行 showmount -e + 主机IP 测试和nfs服务器是否能够联通
```
apt install nfs-common

showmount -e 192.168.1.93
```
如果看到返回：
```
Export list for 192.168.1.93:
/nfs *
```
说明文件服务器是ok的。

将该目录挂载到本地，每个机器都需要挂载，包括管理节点的机器。为的是保证每台机器都能够从/mnt/nfs/目录获取数据。
```
mkdir -p /mnt/nfs
mount 192.168.1.93:/nfs  /mnt/nfs
```
访问本地的/mnt/nfs目录，就可访问服务端共享的目录了。
## 部署 Fabric
### 部署文件准备
进入nfs目录
```
cd /mnt/nfs
```
从github获取示例脚本
```
git clone https://github.com/qiushaoxi/fabric-in-swarm.git
cd fabric-in-swarm
```
每个节点都需要有fabric相关的docker image，我例子使用fabric 1.0.0版本。

如果还没有，需要在每台机器执行下载脚本，拉取image
```
bash download-dockerimages.sh
```
如果觉得下载速度太慢，可以使用阿里云镜像加速，或者在本地部署docker hub，具体方法可以参考[本地Docker Hub配置](http://qiushaoxi.com/2017/12/05/Docker-Hub-Configration/).

### 执行部署命令

设置相关环境变量，FABRIC_IP是你Swarm管理节点的ip
```
export FABRIC_IP=192.168.1.93
```
执行docker stack deploy命令启动stack，docker-compose-cli.yaml是docker-compose v3 配置文件，配置了相关的service。

示例的配置参考fabric官方e2e_cli，1个orderer，4个peer，一个cli。命令最后的 e2e 是stack的名称。
```
cd e2e_cli
docker stack deploy -c docker-compose-cli.yaml e2e
```
docker-compose文件中配置了cli容器会自动执行脚本，执行创建通道、加入通道等一系列操作。

docker service 会根据每台机器的性能和负载，选择启动container的机器。

![运行情况](/img/fabric-in-swarm/cluster.png)

### 查看执行结果
执行下面命令，查询执行日志。
```
docker service logs -f e2e_cli
```

如果看不到内容，（我在本地可以用这个命令，阿里云看不到，原因不明）。执行
```
docker service ps e2e_cli
```
可以看到cli容器的id和运行在哪台机器上。

到运行cli容器的机器上执行
```
docker logs [cli_container_id]
```
查看执行日志。

如果最后出现 END-E2E，则说明执行成功。

### 移除 stack
执行
```
docker stack rm e2e
```
就可以停止并移除所有配置启动的service。

### 其他说明
查看stack
```
docker stack ls
```
查看service
```
docker service ls
```
因为运行chaincode的container是通过unix://var/run/docker.sock启动的，不是由swarm启动的，所以通过docker service 是看不到的。需要到具体执行的机器上执行
```
docker ps
```
才能看到。

而且，就算是停止了stack也不会自动删除这些container，不过因为peer停止了，这些chaincode的container也会自动停止。

## 参考文献
[Docker Compose 配置文件详解（V3）](http://www.jianshu.com/p/748416621013)

[官方接口文档](https://docs.docker.com/compose/compose-file/
)

[多主机网络下 Docker Swarm 模式的容器管理](http://blog.csdn.net/u014743697/article/details/53004638
)

[ubuntu 16.04 nfs服务的搭建](https://www.cnblogs.com/MoreExcellent/p/7222895.html)