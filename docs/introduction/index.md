# 介绍
***
波场 (TRON) 是一个支持智能合约的开源公共区块链平台。
波场与以太坊兼容，这意味着可以将以太坊上的智能合约直接迁移到波场上，或者稍作修改即可。
波场依靠独特的共识机制实现了远超以太坊的高 TPS，为开发者带来了更快的交易速度和良好的用户体验。

波场与以太坊的主要区别体现在以下几个方面：

- **共识机制：** 以太坊目前采用 PoW 共识机制，未来将转向 PoS 机制。波场采用 DPoS 共识机制。有关波场共识机制的更多信息，请参阅[波场共识机制](../tron-protocol/consensus.md)。
- **资源模型：** 在以太坊上进行交易需要支付 `gas` 费用，而波场网络的交易则消耗带宽和能量资源。带宽用于衡量交易大小，以字节为单位；交易越大，消耗的带宽资源越多。能量用于衡量 TRON 虚拟机 (TVM) 在波场网络上执行特定操作所需的计算量。能量的计算方法与以太坊类似，指令执行得越多，消耗的能量越多。有关带宽和能量的详细信息，请参阅[波场资源模型](../tron-protocol/resource-model/index.md)。
- **TVM：** 波场的 TVM 与以太坊的 EVM 兼容，但在某些细节上存在差异，请参阅 [TVM 和 EVM 的差异](../tron-protocol/tvm/index.md)。
- **API：** 以太坊支持 JSON-RPC 2.0 规范 API，波场支持 HTTP 和 gRPC API，还提供与以太坊兼容的 JSON-RPC 2.0 API。

## 概述

本文档旨在帮助您在波场上构建 Web3 应用程序，包括波场的基本概念、核心模块介绍、开发工具和各种示例。

- [对于 DApp 开发人员](#dapp)
- [对于超级代表和选民](#_7)
- [交易所/钱包与 TRON 网络集成](#tron)

## 对于 DApp 开发人员

如果有以太坊开发经验，那么很容易掌握波场的开发。
波场的智能合约开发语言是 Solidity。
开发工具与以太坊上常用的工具（如 Truffle、Remix 和 Web3.js）类似，因此只需花费很少的时间即可熟练使用。

### 工具

以下是波场中开发和部署智能合约的工具：

- [TronBox](https://github.com/tronprotocol/tronbox)：用于编译和部署智能合约的 CLI 工具，类似于以太坊的 Truffle。
- [Tron-IDE](https://www.tronide.io/)：用于编译和部署智能合约的图形用户界面工具，类似于以太坊的 Remix。
- [TronWeb](https://tronweb.network/docu/docs/intro/)：支持波场的 JavaScript SDK，类似于以太坊的 Web3.js。

### 钱包

与 MetaMask 类似，可以通过 TronLink 连接 DApp。TronLink 支持 Chrome 浏览器、Android 和 iOS。

- 与 TronLink 集成的 DApp

### 教程

如果有在波场上开发 DApp 的经验，以下教程可能会有所帮助。
它包括从合约编译、用户界面交互到部署和启动的完整流程。
通过学习构建去中心化应用，开发者可以轻松掌握在波场上部署 DApp 的要领。

- [构建 WEB3 应用](./build-a-web3-app.md)

### 测试网

将 DApp 部署到 Shasta 和 Nile 测试网以及波场主网。
要了解更多信息，请查看[网络](../tron-protocol/networks.md)。

## 对于超级代表和选民

超级代表是网络中的参与者，他们运行完整节点来生成区块，总共有 27 个节点。
超级代表通过投票选举产生，负责波场的区块验证和生成。
此外，超级代表还负责网络的治理，这对网络的健康运行至关重要。

选民通过质押 TRX 获得投票权和资源（能量或带宽），并使用投票权选举超级代表，同时获得奖励。

- [成为超级代表](../tron-protocol/super-representatives/index.md)
- 运行全节点
- [奖励](../tron-protocol/super-representatives/index.md)
- [经纪比率](../tron-protocol/super-representatives/index.md)
- [委员会和提案](../tron-protocol/super-representatives/index.md)

## 交易所/钱包与 TRON 网络集成

如果经营一家交易所或提供钱包服务，可以参考此处了解如何与波场进行整合。