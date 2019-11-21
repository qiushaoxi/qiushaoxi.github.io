---
layout: post
title: "使用 NodeJS 调用 API 接口 进行转账和 Bitshares 内盘交易"
subtitle: "bitsharesjs-sample"
date: 2018-01-25 09:55:00
author: "Joshua"
categories:
  - 区块链技术
tags:
  - BitShares
  - 区块链
---

使用官方的 bitsharesjs 库进行开发，参考了官方的转账例子，补充了下单的部分，更多的使用细节需要自己研究代码。

<!-- more -->

这里附上转账和下单的示例函数。

```js
/* import {Apis} from "bitsharesjs-ws";
import {ChainStore, FetchChain, PrivateKey, TransactionHelper, Aes, TransactionBuilder} from "bitsharesjs"; */
const Apis = require("bitsharesjs-ws").Apis;
const ChainStore = require("bitsharesjs").ChainStore;
const FetchChain = require("bitsharesjs").FetchChain;
const PrivateKey = require("bitsharesjs").PrivateKey;
const TransactionHelper = require("bitsharesjs").TransactionHelper;
const Aes = require("bitsharesjs").Aes;
const TransactionBuilder = require("bitsharesjs").TransactionBuilder;

const BTS_PRECISION = 100000;
const CNY_PRECISION = 10000;

const wss_url = "wss://bitshares-api.wancloud.io/ws";
var privKey = "privKey"; //change to your privKey
let pKey = PrivateKey.fromWif(privKey);

const transfer = function(fromAccount, toAccount, sendAmount, memo) {
  return new Promise((resolve, reject) => {
    Apis.instance(wss_url, true).init_promise.then(res => {
      console.log("connected to:", res[0].network_name, "network");

      ChainStore.init().then(() => {
        let memoSender = fromAccount;

        Promise.all([
          FetchChain("getAccount", fromAccount),
          FetchChain("getAccount", toAccount),
          FetchChain("getAccount", memoSender),
          FetchChain("getAsset", sendAmount.asset),
          FetchChain("getAsset", sendAmount.asset)
        ]).then(res => {
          // console.log("got data:", res);
          let [fromAccount, toAccount, memoSender, sendAsset, feeAsset] = res;

          // Memos are optional, but if you have one you need to encrypt it here
          let memoFromKey = memoSender.getIn(["options", "memo_key"]);
          console.log("memo pub key:", memoFromKey);
          let memoToKey = toAccount.getIn(["options", "memo_key"]);
          let nonce = TransactionHelper.unique_nonce_uint64();

          let memo_object = {
            from: memoFromKey,
            to: memoToKey,
            nonce,
            message: Aes.encrypt_with_checksum(pKey, memoToKey, nonce, memo)
          };

          let tr = new TransactionBuilder();

          tr.add_type_operation("transfer", {
            fee: {
              amount: 0,
              asset_id: feeAsset.get("id")
            },
            from: fromAccount.get("id"),
            to: toAccount.get("id"),
            amount: {
              amount: sendAmount.amount,
              asset_id: sendAsset.get("id")
            },
            memo: memo_object
          });

          tr.set_required_fees().then(() => {
            tr.add_signer(pKey, pKey.toPublicKey().toPublicKeyString());
            console.log("serialized transaction:", tr.serialize());
            tr.broadcast(() => {
              console.log("transaction done");
            });
          });
        });
      });
    });
  });
};

const transferBTS = function(fromAccount, toAccount, amount, memo) {
  transfer(
    fromAccount,
    toAccount,
    { amount: amount * BTS_PRECISION, asset: "BTS" },
    memo
  );
};

const create_order = function(orderAccount, sellAmount, buyAmount) {
  return new Promise((resolve, reject) => {
    Apis.instance(wss_url, true).init_promise.then(res => {
      console.log("connected to:", res[0].network_name, "network");

      ChainStore.init().then(() => {
        Promise.all([
          FetchChain("getAccount", orderAccount),
          FetchChain("getAsset", sellAmount.asset),
          FetchChain("getAsset", buyAmount.asset)
        ]).then(res => {
          // console.log("got data:", res);
          let orderAccount = res[0];
          let sellAsset = res[1];
          let feeAsset = sellAsset;
          let buyAsset = res[2];

          let tr = new TransactionBuilder();

          tr.add_type_operation("limit_order_create", {
            fee: {
              amount: 0,
              asset_id: feeAsset.get("id")
            },
            seller: orderAccount.get("id"),
            amount_to_sell: {
              amount: sellAmount.amount,
              asset_id: sellAsset.get("id")
            },
            min_to_receive: {
              amount: buyAmount.amount,
              asset_id: buyAsset.get("id")
            },
            expiration: Math.floor(Date.now() / 1000) + 20,
            fill_or_kill: false
          });

          tr.set_required_fees().then(() => {
            tr.add_signer(pKey, pKey.toPublicKey().toPublicKeyString());
            console.log("serialized transaction:", tr.serialize());
            tr.broadcast(() => {
              console.log("transaction done");
            }).catch(err => {
              console.log(err);
            });
          });
        });
      });
    });
  });
};

transferBTS("imba", "qiushaoxi", 0.05, "test nodejs");
create_order(
  "imba",
  { amount: 0.5 * BTS_PRECISION, asset: "BTS" },
  { amount: 5 * CNY_PRECISION, asset: "CNY" }
);
```
