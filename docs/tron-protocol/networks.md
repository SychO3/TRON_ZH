# 网络
***
由于 TRON 是一个协议，这意味着可以有多个独立的`网络`符合该协议，但它们之间并不相互影响。

网络是您可以访问的不同 TRON 环境，用于开发、测试或生产用例。您的 TRON 账户可以在不同的网络中使用，但您的账户余额和交易历史不会从 TRON 主网络中转移。出于测试目的，了解哪些网络可用以及如何获取测试网络 TRX 是非常有用的，这样您就可以玩转它。

## 公共网络
世界上任何拥有互联网连接的人都可以访问公共网络。任何人都可以读取或创建公共区块链上的交易，并验证正在执行的交易。

### 主网
Mainnet 是 TRON 生产区块链的主要公共区块链，分布式账本上的实际价值交易就发生在这里。当人们和交易所讨论 TRX 价格时，他们谈论的是主网 TRX。

- Browser：https://tronscan.org
- TronGrid API：https://api.trongrid.io
- TronGrid json-rpc API: https://api.trongrid.io/jsonrpc
- TronGrid gRPC fullnode API: grpc.trongrid.io:50051
- TronGrid gRPC solidity API: grpc.trongrid.io:50052
- Database backup：Data backup

#### 基础设施提供商
除了 TRON Grid 的 RPC 服务外，您还可以使用其他基础设施提供商的 RPC 服务：

- Ankr
- GetBlock
- QuickNode
- NOWNodes

#### 公共节点
公共节点是稳定的在线主网节点，可用作 TRON 网络中的种子节点，用于发现节点：

- 3.225.171.164
- 52.53.189.99
- 18.196.99.16
- 34.253.187.192
- 18.133.82.227
- 35.180.51.163
- 54.252.224.209
- 52.15.93.92
- 34.220.77.106
- 15.207.144.3
- 13.124.62.58
- 15.222.19.181
- 18.209.42.127
- 3.218.137.187
- 34.237.210.82
- 13.228.119.63
- 18.139.193.235
- 18.141.79.38
- 18.139.248.26

公共节点的端口：

- HTTP port : 8090
- HTTP solidity port : 8091
- GRPC port: 50051
- GRPC solidity port : 50061
- P2P network port: 18888


### 测试网
除了 Mainnet，还有公共测试网。这些网络由协议开发者或智能合约开发者使用，用于在部署到 Mainnet 之前测试协议升级和潜在的智能合约。

一般来说，在部署到主网上之前，必须在测试网上测试您编写的任何合约代码。测试网上的 TRX 没有实际价值；因此，测试网 TRX 没有市场。任何人都可以从龙头获取测试网 TRX。

#### Shasta
Shasta 测试网络的参数与主网络一致。目前，Shasta 测试网络不支持添加任何人运行的新节点。

- Website:https://www.trongrid.io/shasta
- Faucet:https://shasta.tronex.io
- Browser:https://shasta.tronscan.org
- http API: https://api.shasta.trongrid.io
- grpc fullnode API: grpc.shasta.trongrid.io:50051
- grpc solidity API: grpc.shasta.trongrid.io:50052
- json-rpc API: https://api.shasta.trongrid.io/jsonrpc

#### Nile
Nile 测试网用于测试 TRON 的新功能，代码版本一般领先于主网。

- Website：http://nileex.io
- Faucet: http://nileex.io/join/getJoinPage
- Browser: https://nile.tronscan.org
- Status: http://nileex.io/status/getStatusPage
- http API: https://api.nileex.io/
- Trongrid http AP: https://nile.trongrid.io/
- grpc API: grpc.nile.trongrid.io:50051
- grpc fullnode API: grpc.nile.trongrid.io:50051
- grpc solidity API: grpc.nile.trongrid.io:50061
- Database backup：http://47.90.243.177

#### Tronex
Tronex 主要用于 sun-network 测试。

- Website: http://testnet.tronex.io
- Faucet:http://testnet.tronex.io/join/getJoinPage
- Browser:http://3.14.14.175:9000
- Status:http://testnet.tronex.io/status/getStatusPage
- Full Node API: https://testhttpapi.tronex.io
- Event API: https://testapi.tronex.io
- Public Fullnode：
    - 47.252.87.28
    - 47.252.85.13
- Database backup: http://47.252.81.247


## 私有网络
如果节点没有连接到公共网络（主网或测试网），则为私有网络，请参阅如何构建私有链。

在将 TRON Dapp 部署到主网之前，您可以在专用网络上进行开发和测试。
与在本地部署网络开发环境类似，您也可以在本地部署私有链来测试 DAPP。
与公共测试网络相比，本地专用网络将提供更快的交互速度。

相关 DAPP 开发工具，请参阅 DAPP 开发工具。

















