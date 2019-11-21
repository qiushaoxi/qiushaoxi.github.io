---
layout: post
title: "Bitshares 私链部署witness节点"
subtitle: "Bitshares setip witness node"
date:       2017-12-20 09:55:00
author: "Joshua"
categories:
  - 区块链技术
tags:
  - BitShares
  - 区块链
---

# 前言

单节点私链搭建请参考 剑有偏峰 的文章：

[比特股环境搭建](http://www.jianshu.com/p/b54782cd1926)

[编译比特股网页钱包](http://www.jianshu.com/p/5be0344e30cd)

[搭建比特股的水龙头注册服务](http://www.jianshu.com/p/a89b3835d4e8)

# 多节点部署

通过之前的文章，部署了单节点的 Bitshares 区块链，接下去模拟真实场景，应该是有很多区块链节点才能实现去中心化高可用的架构。

<!-- more -->

在另一台机器上同样编译 Bitshares-core ，将原来的 genesis.json 复制到本机的 bitshares-core/programs/witness_node 目录下，执行

```
./witness_node   -d test_net --seed-nodes "[]" --genesis-json "genesis.json"
```

会初始化目录 test_net
然后

```
ctrl-c
```

退出程序，编辑 test_net/config.ini，修改如下几项配置

```
p2p-endpoint = 0.0.0.0:9090
seed-nodes = ["192.168.23.248:9090"]
rpc-endpoint = 0.0.0.0:8090
enable-stale-production = true
```

其中 seed-nodes 填写之前启动的区块链节点的 ip 和 p2p-endpoint 端口

再次启动 witness 程序

```
./witness_node   -d test_net
```

这样就成功部署了另一个区块链节点，但是由于没有 witness 权限，所以不会出块，只会同步区块信息。

从 github 下载下来的代码启动私链，默认的 witness 是 init0 ～ inti10。

# 成为 witness

这里默认读者已经会使用 cli_wallet .

启动 cli_wallet ，执行：

```
suggest_brain_key
```

获得新账户信息

```
{
  "brain_priv_key": "LYSSA JUBILEE GIDDIFY KEMPITE INULASE SOBBER GROVEL ENCLAVE T SAHME MYXA GLIDE OVERALL SYNC GROWLER THRILL",
  "wif_priv_key": "5Jiie6zMJWAJ6P8QmivSVjfbPyWa9uKNtwtFXQC3TBoGZDnKXwN",
  "pub_key": "BTS7Zpu3gZmVAGAXgGbD4CzJBwP9uFWtwN5V83TnJQsF9eZ6CnSMx"
}
```

如果之前没有 import nathan 用户，执行：

```
import_key nathan 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

nathan 如果看过之前帖子应该知道是什么。

通过生成的 pub_key 注册用户 joshua（任意名称）

```
register_account joshua "BTS7Zpu3gZmVAGAXgGbD4CzJBwP9uFWtwN5V83TnJQsF9eZ6CnSMx" "BTS7Zpu3gZmVAGAXgGbD4CzJBwP9uFWtwN5V83TnJQsF9eZ6CnSMx" nathan nathan 50 true
```

从 nathan 账户转一部分 BTS 到新建的用户

```
transfer nathan joshua 2000000000 BTS "here is some cash" true
```

导入用户的私钥到 cli 钱包，私钥是刚才生成内容的 wif_priv_key 字段

```
import_key joshua 5Jiie6zMJWAJ6P8QmivSVjfbPyWa9uKNtwtFXQC3TBoGZDnKXwN
```

升级用户成为终身成员

```
upgrade_account joshua true
```

创建 witness 用户

```
create_witness joshua "" true
```

为新建的 witness 用户投票，

```
vote_for_witness joshua joshua true true
```

获取 witness 公私钥

```
get_witness joshua
```

得到结果：

```
{
  "id": "1.6.12",
  "witness_account": "1.2.18",
  "last_aslot": 52765,
  "signing_key": "BTS6UyKzfSim5anCFKZsWB8HPtBSwt8yUthiFCE6oaTXo9Qf6z85e",
  "pay_vb": "1.13.9",
  "vote_id": "1:22",
  "total_votes": "898790081908868",
  "url": "",
  "total_missed": 13,
  "last_confirmed_block_num": 15402
}
```

witness 的公钥就是"signing_key"字段，记录下 witness_account ，后面配置有用，再执行

```
dump_private_keys
```

获取目前钱包中存储的私钥

```
[[
    "BTS78CuY47Vds2nfw2t88ckjTaggPkw16tLhcmg4ReVx1WPr1zRL5",
    "5JDh3XmHK8CDaQSxQZHh5PUV3zwzG68uVcrTfmg9yQ9idNisYnE"
  ],[
    "BTS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV",
    "5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"
  ],[
    "BTS7Zpu3gZmVAGAXgGbD4CzJBwP9uFWtwN5V83TnJQsF9eZ6CnSMx",
    "5Jiie6zMJWAJ6P8QmivSVjfbPyWa9uKNtwtFXQC3TBoGZDnKXwN"
  ],[
    "BTS6UyKzfSim5anCFKZsWB8HPtBSwt8yUthiFCE6oaTXo9Qf6z85e",
    "5JM1AsnRJDGDa8x7NR3GtNDhQzzNi6VEs9XmXyuDhLEnJDUwnS8"
  ]
]
```

根据"signing_key"获取对应的私钥，就是：

```
["BTS6UyKzfSim5anCFKZsWB8HPtBSwt8yUthiFCE6oaTXo9Qf6z85e","5JM1AsnRJDGDa8x7NR3GtNDhQzzNi6VEs9XmXyuDhLEnJDUwnS8"]
```

记录好这个公私钥对，接下去修改 test_net/config.ini

```
# ID of witness controlled by this node (e.g. "1.6.5", quotes are required, may specify multiple times)
witness-id = "1.6.2"
witness-id = "1.6.3"
witness-id = "1.6.12"

# Tuple of [PublicKey, WIF private key] (may specify multiple times)
private-key = ["BTS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV","5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"]
private-key = ["BTS6UyKzfSim5anCFKZsWB8HPtBSwt8yUthiFCE6oaTXo9Qf6z85e","5JM1AsnRJDGDa8x7NR3GtNDhQzzNi6VEs9XmXyuDhLEnJDUwnS8"]
```

增加新建的 witness-id 和对应的公私钥，重启 witness 重需

```
./witness_node   -d test_net
```

_这一部分就完成了_

不过因为默认的更新 witness 的时间比较长，24 小时，所以要等 24 小时才能看到 joshua 成为 witness，如果想快一点，需要重头开始做，在执行

```
./witness_node --create-genesis-json "genesis.json"
```

之后，编辑 genesis.json，修改下面这个参数为 600，这样 10 分钟就会更新 witness

```
"maintenance_interval": 600,
```

---

![示例图](http://upload-images.jianshu.io/upload_images/7302525-95ffade6d5ccc8e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
