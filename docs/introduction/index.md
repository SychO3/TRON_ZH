# 介绍
***
波场是一个开源的公共区块链平台，支持智能合约。
波场兼容以太坊，这意味着可以将以太坊上的智能合约直接或经过少量修改后迁移到波场。
波场依赖其独特的共识机制实现了波场网络远超以太坊的高每秒交易数（TPS），为开发者带来了更快速交易的良好体验。

波场与以太坊的不同之处主要体现在以下几个方面：

- **共识机制：** 当前，以太坊网络采用工作量证明（POW）共识机制，并将在未来采用权益证明（POS）共识机制。波场的共识机制是委托权益证明（DPOS）。有关波场共识机制的更多信息，请参阅[波场共识机制](../tron-protocol/consensus.md)。
- **资源模型：** 以太坊交易需要支付 Gas 费用，而波场网络交易需要支付带宽和能量费用。其中，带宽是用于衡量交易大小的单位，单位为字节。交易越大，消耗的带宽资源就越多。能量是衡量在波场网络上，波场虚拟机（TVM）执行特定操作所需计算量的单位。能量的计算方式与以太坊相同。交易执行的指令越多，消耗的能量就越多，并且不同指令消耗的能量也各不相同。有关带宽和能量的更多信息，请参阅[波场资源模型](../tron-protocol/resource-model/index.md)。
- **TVM：** 波场虚拟机（TVM）和以太坊虚拟机（EVM）兼容，但在一些细节上有所不同。详情请参阅[《TVM 和 EVM 的区别》](../tron-protocol/tvm/index.md)。
- **API：** 以太坊支持 JSON-RPC 2.0 规范的 API，波场支持 Http 和 gRPC API，并且波场还提供与以太坊兼容的 JSON-RPC 2.0 API。

## 概述

本文档旨在帮助您在波场上构建 Web3 应用程序，包括波场的基础概念和核心模块、开发工具及各种示例的介绍。您可以根据需要选择主题：

- [对于 DApp 开发人员](#dapp)
- [对于超级代表和选民](#_7)
- [交易所/钱包与 TRON 网络集成](#_8)

## 对于 DApp 开发人员

如果您具备以太坊的开发经验，那么您将很容易掌握波场的开发。
波场的智能合约开发语言是 Solidity。
其开发工具与您在以太坊上熟悉的工具（例如 Truffle、Remix 和 Web3js）相似，您可以在很短的时间内熟练使用这些工具。

### 工具

以下是波场中开发和部署智能合约的工具：

- [TronBox](https://github.com/tronprotocol/tronbox)：用于编译和部署智能合约的 CLI 工具，类似于以太坊的 Truffle。
- [Tron-IDE](https://www.tronide.io/)：用于编译和部署智能合约的图形用户界面工具，类似于以太坊的 Remix。
- [TronWeb](https://tronweb.network/docu/docs/intro/)：支持波场的 JavaScript SDK，类似于以太坊的 Web3.js。

### 钱包

与 MetaMask 类似，可以通过 TronLink 连接 DApp。TronLink 支持 Chrome 浏览器、Android 和 iOS。

- 与 TronLink 集成的 DApp

### 教程

如果您在波场上开发 DApp 的经验为零，您可能会发现以下教程既友好又有帮助。
该教程包括从编译合约、用户界面交互到部署和启动的完整流程。
通过学习构建去中心化图书馆，开发人员可以轻松掌握如何在波场网络上部署自己的 DApp。

- [构建 WEB3 应用](./build-a-web3-app.md)

### 测试网

将 DApp 部署到 Shasta 和 Nile 测试网以及波场主网。
要了解更多信息，请查看[网络](../tron-protocol/networks.md)。

## 对于超级代表和选民

超级代表是波场网络中的参与者，负责运行完整节点以进行区块生产，总共有27个超级代表。
这些超级代表通过投票选举产生，并负责波场网络的区块验证和生成。
此外，超级代表还负责网络治理，对网络的健康运行起到至关重要的作用。

投票者通过质押 TRX 来获得投票权和资源（能量或带宽），获得的投票权可以用来投票选出超级代表，同时还能获得奖励。

- [成为超级代表](../tron-protocol/super-representatives/index.md)
- 运行全节点
- [奖励](../tron-protocol/super-representatives/index.md)
- [经纪比率](../tron-protocol/super-representatives/index.md)
- [委员会和提案](../tron-protocol/super-representatives/index.md)

## 交易所/钱包集成波场网络

如果您运营交易所或提供钱包服务，您可以参考此处以便与波场进行集成。