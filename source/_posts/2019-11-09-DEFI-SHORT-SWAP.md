---
layout: post
title: "基于三角Swap的去中心化做空机制"
date: 2019-11-09 09：00：00
author: "Tiny Hill"
categories:
  - 区块链技术
tags:
  - 区块链
  - Defi
  - 去中心化金融
  - dex
  - 去中心化交易所
  - uniswap
---

> 无抵押、不爆仓的做空机制是否可能？

# 背景

做空是最常用的金融工具之一，而目前还没有出现成熟的去中心化做空的机制。其中一个重要的原因在于预言机机制不成熟，导致爆仓的时效性无法得到保证。

本文尝试讨论一种无需抵押和爆仓的做空机制，这样就可以不受目前预言机等技术限制的影响。

<!-- more -->

该想法最早来源于 aiden pearce 的算法稳定币 VUSD 的设计。但是，我认为他用错了地方，算法稳定币目前来看，还是空中楼阁，看不到实现的可能。但是这个机制用在去中心化交易所的做空机制上，可能是个不错的尝试。

# triswap

uniswap 是一个经过时间考验，非常成熟好用的去中心化交易协议。它通过交易对的底仓形成价格和深度，为交易者提供即刻的流动性。

假设一个基于 uniswap 模型的 EOS/USDT 交易对，在交易对中引入一个做空权益代币，short-EOS，简称 sEOS。在对底仓注入交易对两边的资金同时，根据 EOS/sEOS 的价格发行相同价值的做空权益 sEOS。让交易对中的 EOS、sEOS 和 USDT 形成三角的 uniswap，这里简称 triswap。

3 个币种会自然形成 3 个交易对，分别是 EOS/USDT、sEOS/USDT、EOS/sEOS，根据 uniswap 的模型，价格来源于币种在底仓中余额的比值，而这 3 个交易对共享底仓中 3 个余额。所以每个交易对发生交易后，对其他 2 个交易对的价格也会产生直接的影响。

## 举例

- 初始 0 底仓，交易对补充 10000 USDT 和 5000 EOS 形成交易对。
- EOS/sEOS 交易对在实际情况中会有市场价格，在初始阶段，需要对 sEOS 拟定一个价格
- 例子假设 EOS/sEOS = 1/10
- 补充底仓的同时新增发行同等价值的 50000 sEOS
- 初始价格：EOS：2 U ， sEOS：0.2 U
- 以下举例都由最初的底仓展开，交易资金量为底仓 1%
- 3 个交易对有 6 中交易行为

### USDT->EOS

100 USDT -> 49.5 EOS

交易后：

- EOS/USDT： 2.04， 上涨 4%
- sEOS/USDT： 0.202，上涨 2% （同时上涨）
- sEOS/EOS： 0.099，下降 1%

### EOS->USDT

50 EOS -> 99.01 USDT

交易后：

- EOS/USDT： 1.96， 下跌 4%
- sEOS/USDT： 0.198， 下跌 2% （同时下跌）
- sEOS/EOS： 0.101， 上涨 1%

### USDT->sEOS

100 USDT -> 495.05 sEOS

交易后：

- EOS/USDT： 2.02， 上涨 2%
- sEOS/USDT：0.0204，上涨 4%
- sEOS/EOS： 0.101， 上涨 1%

### sEOS->USDT

_初始情况不存在可以卖出的 sEOS，此处假设_

500 sEOS -> 99.01 USDT

交易后：

- EOS/USDT： 1.98， 下跌 2%
- sEOS/USDT：0.196，下跌 4%
- sEOS/EOS： 0.099，下降 1%

### EOS->sEOS

50 EOS-> 495.05 sEOS

交易后：

- EOS/USDT： 1.98 ，下降 1%
- sEOS/USDT：0.202，上涨 1%
- sEOS/EOS： 0.102，上涨 2%

### sEOS->EOS 卖出平空

_初始情况不存在可以卖出的 sEOS，此处假设_

500 sEOS-> 49.5 EOS

交易后：

- EOS/USDT： 2.02 ， 上涨 1%
- sEOS/USDT：0.0198 ，下跌 1%
- sEOS/EOS： 0.098 ， 下跌 2%

# 做空逻辑

- 认为 EOS 价格会下跌，用 EOS 买入 sEOS
- EOS/USDT 价格下跌，普通用户、套利用户会卖出 EOS
- 如果交易 EOS->USDT，sEOS/USDT 价格下跌幅度为 EOS/USDT 的一半，sEOS 相对 EOS 的价格上涨
- 如果交易 EOS->sEOS，同样能够使 EOS/USDT 下降以匹配价差，此时，sEOS 相对 EOS 价格的涨幅是交易额的 2 倍
- 认为下跌行情结束，用 sEOS 买回 EOS，可以获得相对购买时更多的 EOS

# 特点

- sEOS 无需其他资产抵押
- sEOS 没有爆仓
- sEOS 只有最后兑现成 EOS 才能完成做空的盈利，sEOS 对 USDT 不会产生做空盈利
- 这是一个赚币的做空方案
- 无杠杆，波动相对小

# 产品化方案

- 在 uniswap 的币币兑换的基础上增加“币做空”
- “币做空”交易 EOS/sEOS
- 考虑到 sEOS/USDT 对于 EOS/USDT 并不是反向的作用，可以禁用 USDT->sEOS ，只允许 sEOS->USDT 的权益兑现

# 最后

全文以 USDT、EOS、sEOS 举例，可以适用于任何交易对。但是在 unsiwap 多交易对拼接形成的交易对中，可能效果会不同。

sEOS 的发行是对底仓注资的同时。所以，在底仓取回的同时，也会根据当时的 EOS/sEOS 价格销毁相同价值的 sEOS

EOS/sEOS 的价格，是一个市场博弈的结果，对于什么价格是合理的，我目前也没有想通。如果想通了可以再写一篇。

这个想法还是很粗糙，欢迎大家一起讨论。

文章原创首发[力场](https://lichang.io/articleDetail/931431)
