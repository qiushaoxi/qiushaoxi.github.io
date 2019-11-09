---
layout:     post
title:      "Fabric v1.1 版本新功能简析"
subtitle:   "fabric v1.1 release notes"
date:       2017-11-09 17:55:00
author:     "Joshua"
categories:
  - 区块链技术
tags:
    - Hyperledger
    - 区块链
    - fabric
---

> 本文内容参考官方版本说明：
[v1.1.0-preview](<https://github.com/hyperledger/fabric/releases/tag/v1.1.0-preview>) ，简单看了一下变动，后续如果有时间会写更详细的内容。

官方已经发布v1.1.0-preview版本，V1.1正式版也不远了，来看看这个版本都增加了哪些功能。

## [FAB-2331](<https://jira.hyperledger.org/browse/FAB-2331>) - Node.js Chaincode 支持
增加了Node.js开发Chaincode的支持，原先支持Golang和java开发Chaincode，具体内容可以关注[fabric-chaincode-node](https://github.com/hyperledger/fabric-chaincode-node)项目，开发模型和go差不多，通过shim接口调用。
## [FAB-5363](<https://jira.hyperledger.org/browse/FAB-5363>) - Node.js SDK Connection Profile 
通过配置文件的方式管理SDK的连接。参考原先官方的供的node sdk的sample，也有通过配置文件的方式配置连接地址，但是在sdk的外层。目前看，这个功能应该是sdk自带的属性。

配置文件中包含peer节点、orderer节点的地址，和一些加密信息的配置，基本上使用中所需要的配置内容都集中到这个文件中，通过sdk接口可以方便读取。

以下是简单的例子：

var client = Client.loadFromConfig(json)

var mychannel = client.getChannel('mychannel')

mychannel.sendTransactionProposal()

mychannel.sendTransaction()
## [FAB-830](<https://jira.hyperledger.org/browse/FAB-830>) - Encryption library for chaincode
给Chaincode增加了加密套件，这个东西还是比较重要的，方便用户对上链数据进行灵活的加密。
## [FAB-5346](<https://jira.hyperledger.org/browse/FAB-5346>) - Attribute-based Access Control
基于属性的权限控制。简单说就是在Ecert中加入特定的属性，让chaincode可以根据这个属性进行更加丰富的权限控制。
## [FAB-6089](<https://jira.hyperledger.org/browse/FAB-6089>) - Chaincode APIs to retrieve creator cert info
新增了GetCreator接口方法用来获取交易发起者的信息，方便进行权限控制。
## [FAB-6421](<https://jira.hyperledger.org/browse/FAB-6421>) - Performance improvements
性能优化


## bug修复和其他修改请看[Change Log](<https://github.com/hyperledger/fabric/blob/master/CHANGELOG.md#v110-preview>)
