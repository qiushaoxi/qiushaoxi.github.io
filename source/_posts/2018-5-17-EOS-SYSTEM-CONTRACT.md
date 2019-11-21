---
layout: post
title: "EOSIO Dawn 4.0 系统合约解读"
subtitle: ' "EOSIO Dawn 4.0 System Contract Analysis"'
date: 2018-5-17 13:07:00
author: "Joshua"
categories:
  - 区块链技术
tags:
  - 区块链
  - EOS
---

Dawn 4.0 发布后，基本上白皮书上提到的特性都得到了实现，特别是投票、资源抵押和内存买卖。

EOS 的网络带宽和计算资源是通过抵押代币的方式获得，如果不需要了可以解除抵押收回原先的代币。

投票的权重和抵押代币所获得的带宽和计算资源相关，抵押了越多，投票的权重越大。同时投票的权重会随着时间的推移减少，这个设定鼓励用户持续更新自己的投票。

而存储资源则是通过购买来得到，但是并不会消耗，当不再需要存储资源的时候可以出售。但是存储资源相对于 EOS 代币的价格会随着市场对于存储资源的需求而变动。

而这些功能都是在 eosio.system 这个智能合约中实现。

本文试图通过分析 eosio.system 中所有的功能函数来解读 eos 系统功能。

<!-- more -->

## 函数列表

在 eosio.system.hpp 头文件中包含以下函数声明：

### delegatebw

```cpp
void onblock( block_timestamp timestamp, account_name producer );

void delegatebw( account_name from, account_name receiver,asset stake_net_quantity, asset stake_cpu_quantity, bool transfer );

```

delegatebw 函数用来实现抵押代币获取 cpu 和带宽资源。参数定义：

- from : 从哪个账号扣除用来抵押的代币
- receiver : 抵押的代币的接受者，表示抵押获取的资源作用在哪个账号上
- stake_net_quantity : 用来抵押带宽资源的代币数量
- stake_cpu_quantity : 用来抵押计算资源的代币数量
- transfer : 是否接受者可以主动解除抵押获得代币，如果不是，只有发起者能够解除抵押收回代币

### undelegatebw

```cpp
void undelegatebw( account_name from, account_name receiver,asset unstake_net_quantity, asset unstake_cpu_quantity );
```

undelegatebw 函数用来解除抵押，释放资源，收回代币，参数定义：

- from : 解除用哪个账号所抵押的代币
- receiver : 解除作用在哪个账号上的抵押代币
- unstake_net_quantity : 解除用来获取带宽资源的代币数量
- unstake_cpu_quantity : 解除用来获取计算资源的代币数量

解除抵押之后，资源会马上释放，同时投票权重也相应消失。但是代币需要等待一定时间才能回到账号里，目前  的设定是 3 天。

### buyram

```cpp
void buyram( account_name buyer, account_name receiver, asset tokens );

void buyrambytes( account_name buyer, account_name receiver, uint32_t bytes );
```

这两个函数的作用是购买存储资源，区别是买特定数量的代币还是买特定大小的内容。参数定义:

- buyer : 购买存储资源的账号
- receiver : 接受存储资源的账号
- tokens : 购买存储资源所用的代币数量
- bytes : 都买存储资源空间大小的数值

### sellram

```cpp
void sellram( account_name receiver, int64_t bytes );
```

sellram 函数功能是出售不需要的存储资源。参数定义：

- receiver : 出售资源代币的接受账号
- bytes : 出售多少空间的存储资源

出售后资源会马上释放，收入的代币也会马上入账。

### refund

```cpp
void refund( account_name owner );
```

refund 函数 在 undelegatebw 函数解除抵押后调用，作用是把抵押的代币退回账户。

### regproducer

```cpp
void regproducer( const account_name producer, const public_key& producer_key, const std::string& url, uint16_t location );
```

regproducer 函数的作用是注册成为超级节点候选人。参数定义：

- producer : 候选节点的账户名
- producer_key : 候选节点的账户公钥
- url : 候选节点的网站地址
- location : 候选节点的机房地理位置

注册成为候选人后就可以接受用户的投票了。

### unregprod

```cpp
void unregprod( const account_name producer );
```

unregprod 函数的作用的取消成为候选人。

### voteproducer

```cpp
void setram( uint64_t max_ram_size );

void voteproducer( const account_name voter, const account_name proxy, const std::vector<account_name>& producers );
```

voteproducer 函数的作用是投票。参数定义：

- voter : 投票人
- proxy : 代理投票人
- producers : 得票人列表

有两种投票模式，代理模式和直接投票模式。代理模式是将投票权重委托给一个代理人，让他帮你投票。直接投票模式就是直接投票给你信任的超级节点，最多 30 个。

### regproxy

```cpp
void regproxy( const account_name proxy, bool isproxy );
```

regproxy 函数的作用是注册成为代理人，接受其他用户的委托。

### claimrewards

```cpp
void claimrewards( const account_name& owner );
```

claimrewards 函数的作用是支付超级节点的奖励。

```cpp
void setpriv( account_name account, uint8_t ispriv );
```

## 测试方法

开发者可以通过 cleos system 子命令来测试这些功能。

如果要加入一个测试网络成为出块人，步骤应该是这样：

- 注册账号
- 启动节点同步区块
- 注册成为候选人
- 抵押代币获得资源和投票权重
- 投票给自己
- 当得票足够后，等待一个周期，就可以成为出块人了。
