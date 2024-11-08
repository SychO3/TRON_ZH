# 事件订阅
***


## 背景

在 3.5 版本中，波场开发团队提供了事件订阅机制，因此开发者可以通过事件插件获取链上触发的事件。

## 服务

波场提供以下方式获取事件：

1. TronGrid 封装了事件插件接口，并提供了一个公共、用户友好的 HTTPS 事件查询接口。  
  [https://developers.tron.network/reference/get-events-by-transaction-id](https://developers.tron.network/reference/get-events-by-transaction-id)
2. TronWeb 提供了封装的 JavaScript 方法用于获取事件。
3. 本地设置一个事件插件以提供事件查询。
     - 波场开发团队发布了两种事件插件，Kafka 和 MongoDB 插件。
     - 支持订阅链数据，如区块、交易、合约日志、合约事件等，开发者还可以根据需要自定义插件。
     - 提供事件查询服务 tron-eventquery，提供在线事件查询服务。
4. 使用 Java-tron 的内置消息队列进行事件订阅。

!!! Warning
    在订阅未固化事件时，务必使用两个参数 blockNumber 和 blockHash 作为标准，验证接收的事件是否有效。在如不稳定网络连接导致链重组等特殊情况下，事件重组也可能发生，导致过时事件。

## 插件

插件的功能是实现事件转储。开发者可以根据需要自定义插件，如消息队列、Kafka、MongoDB 或写入本地文件。

要使用事件插件，需要在节点配置文件中将 useNativeQueue 设置为 false，配置如下：

```hocon
event.subscribe = {
    native = {
      useNativeQueue = false // 如果为 true，使用原生消息队列，否则使用事件插件。
      bindport = 5555 // 绑定端口
      sendqueuelength = 1000 // 发送队列的最大长度
    }
   ...
}
```

波场开发团队提供的插件独立于 Java-tron，默认不加载。可以通过配置命令行参数启用。默认情况下，仅支持智能合约事件的订阅。开发者可以通过修改配置文件订阅其他触发器，开发者在定义插件配置文件时具有灵活性，包括消息队列服务器地址、定义的触发器类型等。

## 事件类型

波场事件订阅支持 4 种事件类型：

1. **交易事件**  
    传递给订阅者的参数：
    
       - transactionId: 交易哈希
       - blockHash: 区块哈希
       - blockNumber: 区块编号
       - energyUsage: 能量使用量
       - energyFee: 能量费用
       - originEnergyUsage: 原始能量使用量
       - energyUsageTotal: 总能量使用量

2. **区块事件**  
    传递给订阅者的参数：

     - blockHash: 区块哈希
     - blockNumber: 区块编号
     - transactionSize: 区块中交易的数量
     - latestSolidifiedBlockNumber: 最新固化区块编号
     - transactionList: 交易哈希列表

   3. **合约事件**  
      传递给订阅者的参数：

      - transactionId: 交易 ID
      - contractAddress: 合约地址
      - callerAddress: 合约调用者地址
      - blockNumber: 记录合约相关事件的区块编号
      - blockTimestamp: 区块时间戳
      - eventSignature: 事件签名
      - topicMap: Solidity 语言中的主题映射
      - data: Solidity 语言中的数据信息
      - removed: 'true' 表示日志被移除

   4. **合约日志事件**  
      传递给订阅者的参数：

      - transactionId: 交易哈希
      - contractAddress: 合约地址
      - callerAddress: 合约调用者地址
      - blockNumber: 记录合约相关事件的区块编号
      - blockTimestamp: 区块时间戳
      - contractTopics: Solidity 语言中的主题列表
      - data: Solidity 语言中的数据信息
      - removed: 'true' 表示日志被移除

合约事件和合约日志事件支持事件过滤功能，包括：

- fromBlock: 起始区块编号
- toBlock: 结束区块编号
- contractAddress: 合约地址列表
- contractTopics: 合约主题列表

!!! note
    不支持历史数据查询。  

有关更多详细信息，请参阅以下网址：  
[https://github.com/tronprotocol/TIPs/issues/12](https://github.com/tronprotocol/TIPs/issues/12)