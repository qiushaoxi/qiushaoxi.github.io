---
layout: post
title: "EOSIO随机数的一种实现"
subtitle: ' "A WAY OF EOSIO RANDOM"'
date: 2019-3-11 20:07:00
author: "Joshua"
categories:
  - 区块链技术
tags:
  - 区块链
  - EOS
---

最近帮助 Enumivo（基于 EOSIO）社区做一个投票奖励活动，除了基本的投票奖励之外，还加入了一定的随机机制，增加乐趣。

<!-- more -->

## 合约随机逻辑：

1. 用户发起领取交易，领取投票奖励，同时合约账户转 1ENU 到奖池，同时发起一笔（0-24 小时随机值）的延迟交易。

2. 延迟交易被触发，检查奖池金额，达到了 10ENU，就把 10ENU 都奖励给当前用户。

## 随机机制：

0-24 小时这个时间的随机数，就使用之前常用的利用 block prefix 值来生成，这个不是随机的唯一部分。因为就算用户可以控制随机数来控制延迟时间，也没有办法控制延迟交易发生时奖池的金额，奖池的金额完全是根据其他用户的随机行为来进行累积的。利用这样的随机数方案，或许能够开发一款公平的 Lottery Dapp。

## 可能的攻击方式：

构造交易控制随机数控制延时时间，在延迟交易发生前利用别的账户发起交易，将奖池金额构造到满足开奖条件。像本应用这样金额比较小的开奖，应该没有人会费时费力做这个事情。当奖金足够诱人的时候，就有人和你竞争，增加了开奖的不确定性。

欢迎拍砖。

## 其他信息

- 合约交易历史：http://enumivo.qsx.io/search?q=claimlottery

- 合约开源地址：https://github.com/ENU-Zion/claimforvote

## Enumivo

Enumivo 是一个以 UBI（全民基本收入）为目标的区块链项目

白皮书：

https://enumivo.org/whitepapers/whitepaper_cn.pdf
