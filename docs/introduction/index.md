# 介绍

波场是一个支持智能合约的开源公共区块链平台。
波场与以太坊兼容，这意味着可以将以太坊上的智能合约直接迁移到波场上，或者只需稍加修改即可。
波场依靠独特的共识机制实现了波场网络远超以太坊的高 TPS，为开发者带来了交易速度更快的良好体验。

TRON 与以太坊不同，主要体现在以下几个方面：

- **共识机制：** 目前，以太坊网络采用 POW 共识，未来将采用 POS 共识。
  TRON 的共识机制是 DPOS。有关波场共识机制的更多信息，请参阅[波场共识机制](../tron-protocol/consensus.md)。
- **资源模型：** 以太坊交易需要支付`gas`费，波场网络交易需要支付带宽和能量，
  其中带宽是衡量交易大小的单位，以字节为单位。
  交易越大，消耗的带宽资源就越多。能量是衡量 TVM 在波场网络上执行特定操作所需的计算量的单位。
  能量的计算方法与以太坊相同。交易执行的指令越多，消耗的能量就越多，不同指令消耗的能量也不同。
  有关带宽和能量的更多信息，请参阅[波场资源模型](../tron-protocol/resource-model/index.md)。
- **TVM：** 波场 TVM 和以太坊 EVM 兼容，但在某些细节上存在差异，请参阅 [TVM 和 EVM 的差异](../tron-protocol/tvm/index.md)。
- **API：** 以太坊支持 JSON-RPC 2.0 规范 API，波场支持 Http 和 gRPC API，
  波场还提供与以太坊兼容的 JSON-RPC 2.0 API。

## 概述

本文档旨在帮助您在波场上构建 Web3 应用程序，包括波场的基本概念和核心模块介绍、开发工具以及各种示例。
您可以根据自己的需要选择一个主题：

- [对于 DApp 开发人员](#dapp)
- [对于超级代表和选民](#_7)
- [交易所/钱包与 TRON 网络集成](#tron)

## 对于 DApp 开发人员

如果有以太坊开发经验，那么将很容易掌握波场的开发。
波场的智能合约开发语言是 Solidity。
开发工具与在以太坊上熟悉的工具（如 Truffle、Remix 和 Web3js）类似，只需花很少的时间就能熟练使用。

### 工具

以下是在 TRON 中开发和部署智能合约的工具：

- [TronBox](https://github.com/tronprotocol/tronbox)：用于编译和部署智能合约的 CLI 工具，类似于以太坊 Truffle
- [Tron-IDE](https://www.tronide.io/)：用于编译和部署智能合约的图形用户界面工具，类似于以太坊 Remix
- [TronWeb](https://tronweb.network/docu/docs/intro/)：支持波场的 Javascript SDK，类似于以太坊 Web3.js

### 钱包

与 MetaMask 一样，也可以通过 Tronlink 连接 DApp，Tronlink 支持 Chrome、Android 和 IOS。

- 与 TronLink 集成的 DApp

### 教程

如果没有在波场中开发 DApp 的经验，下面的教程可能会有所帮助。
它包括从编译合约、用户界面交互到部署和启动的一整套流程。
通过学习构建去中心化库，开发者可以轻松掌握如何在波场上部署自己的 DApp。

- [构建 WEB3 应用](./build-a-web3-app.md)

### 测试网

可以将 DApp 部署到 Shasta 和 Nile 测试网以及波场主网。要了解更多信息，请查看[网络](../tron-protocol/networks.md)。

## 对于超级代表和选民

超级代表是网络的参与者，运行一个完整节点来生产区块，总共有 27 个节点。
他们通过投票选举产生，负责波场的区块验证和生成。
此外，超级代表还负责网络的管理，这对网络的健康运行起着至关重要的作用。

投票者通过质押 TRX 获得投票权和资源（能源或带宽），获得的投票权可以投票选举超级代表，同时获得奖励。

- [成为超级代表](../tron-protocol/super-representatives/index.md)
- 运行全节点
- [奖励](../tron-protocol/super-representatives/index.md)
- [经纪比率](../tron-protocol/super-representatives/index.md)
- [委员会和提案](../tron-protocol/super-representatives/index.md)

## 交易所/钱包与 TRON 网络集成

如果经营着一家交易所或提供钱包服务，可以参考此处了解与波场的整合情况。

