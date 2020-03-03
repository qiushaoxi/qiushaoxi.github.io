---
layout: post
title: "RBTC 的跨链安全性"
date: 2020-3-3 16:24:00
author: "Tiny Hill"
categories:
  - RSK
tags:
  - RBTC
  - 区块链
---

RBTC 是 RSK 公链的基础代币，1:1 锚定 BTC，那么 RBTC 是如何做到安全和去中心化的？

<!-- more -->

[RSK 文档链接](https://developers.rsk.co/rsk/architecture/security/)

由于 DeFi（去中心化金融）对于智能 BTC 的需求，目前已经有很多不同类型的 BTC 跨链项目在运行了，而 RSK 的 RBTC 也是这条赛道最早的竞争者之一。

我们都知道，BTC 的跨链，实际上就是在 Bitcoin 链上锁定，再在另外一条链上发行同样金额的币，反方向则是销毁和解锁。

## RBTC 的跨链安全性

由于 Bitcoin 不是一个图灵完备的公链，理论上，无法做到完全的去信任跨链。RSK 在 Bitcoin 链上的锁定的 BTC 资产安全由一组可信第三方（简称 STTP）来保证。这些可信第三方组成了 RSK 联合会（Federation），没有一个单独的 STTP 能够操控锁定的 BTC，只有大多数 STTP 通过才能够释放 BTC 资产。当 STTP 监控到 RSK 链上发起的跨链转账 BTC，STTP 多签执行解锁的操作。除此之外，用户在 Bitcoin 链上发起的跨链转账所锁定的 BTC 并不会固定锁定在一个单独的地址中，RSK 锁定的 BTC 是一个大池子，统一管理。

RSK 联合会成员进行锁定和解锁 BTC 是无需人为干预的。加入 RSK 联合会需要首先对成员的节点程序进行审核，确保节点的基础设施能够提供正确的操作，此外 RSK 实验室也开发了针对 STTP 的硬件安全模块来保证私钥的安全。

截至 2019 年 1 月，RSK 联合会由 15 名知名且高度安全的公证人组成。这些领先的区块链公司通过自治协议来安全锁定 BTC。作为报酬，RSK 联合会成员将获得 RSK 产生的交易费用的 1％，以支付硬件和维护成本。联合会成员的更替由智能合约负责，因此对公众开放。更替生效之前有一个强制性的延迟一周。 如果用户不信任新的 RSK 联合会成员，用户可以将 BTC 转移回 Bitcoin 网络。

如果之后 Bitcoin 通过升级添加了特殊的操作码或可扩展性来支持使用 SPV 验证，RSK 将升级跨链的机制来支持完全去中心化的实现。

## 补充

搜索了很多资料，关于 RSK 联合会的信息很少，官网中并没有找到，唯一找到的[文章](https://news.bitcoin.com/rsk-federation-partners-industry-leaders/)中列出了成员包括：

- Bitgo
- Bitpay
- Bitstamp
- Blockchain
- Blockchain Intelligence Group
- Bitfinex
- BTCC
- Jaxx
- Huobi
- OKCoin
- Xapo

确实都是一些业界最知名的公司。

除了 RBTC，其他一些跨链 BTC 的项目，也都是类似的多机构或者单机构的公证人机制。在目前 Bitcoin 非图灵完备的情况下，还有没有可能出现其他方式的跨链？这是一个值得期待的问题。

关于其他 BTC 跨链项目，可以看看(https://www.chainnews.com/articles/969112149833.htm)
