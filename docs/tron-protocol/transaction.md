# 交易
***
交易是账户发出的经过加密签名的指令。账户将发起交易以更新波场网络的状态。
最简单的交易是从一个账户向另一个账户转移 TRX。

改变链状态的交易需要向整个网络广播。任何节点都可以广播交易请求。
超级节点收到事务后，会执行事务并将其包含在一个区块中，然后将该区块传播到整个网络。

只有在超级节点将交易打包成一个区块，并对该区块进行确认后，交易才能最终得到确认。

交易格式如下：

```javascript
{
    "raw_data": 
    {
        "contract": [{<-->}],
        "ref_block_bytes": "c145",
        "ref_block_hash": "c56bd8a3b3341d9d",
        "expiration": 1646796363000,
        "data": "74657374",
        "timestamp": 1646796304152,
        "fee_limit":10000000000
    },
    "signature":["47b1f77b3e30cfbbfa41d795dd34475865240617dd1c5a7bad526f5fd89e52cd057c80b665cc2431efab53520e2b1b92a0425033baee915df858ca1c588b0a1800" ] 
}
```

提交的交易主要包括以下字段：

- `raw_data.contract` - 交易的主要内容，contract 是一个列表，但目前只使用一个元素。不同类型的交易有不同的合约内容。例如，对于 TRX 转帐类型的交易，合约将包括转帐金额、接收地址和其他信息。波场支持多种类型的合约，详情请参阅下面的 交易类型 部分。
- `raw_data.ref_block_bytes` - 交易块的高度，使用参考块高度的第 6 到第 8 个（独占）字节，共 2 个字节。参考区块用于 `TRON TAPOS 机制`，可以防止在不包含参考区块的分叉上重放事务。一般情况下，最新固化的区块被用作参考区块。
- `raw_data.ref_block_hash` - 交易块的哈希值，使用参考区块哈希值的第 8 至第 16 个（独占）字节，共 8 个字节。参考区块用于 `TRON TAPOS 机制`，可防止在不包含参考区块的分叉上重放事务。一般情况下，最新固化的区块被用作参考区块。
- `raw_data.expiration` - 交易过期时间，过期后交易将不再打包。如果交易是通过调用 java-tron API 创建的，节点会自动将其过期时间设置为在节点最新区块的时间戳上加上 60 秒。过期时间间隔可在节点配置文件中修改，最大值不能超过 24 小时。
- `raw_data.data` - 交易备忘录。
- `raw_data.timestamp` - 交易时间戳，设置为交易创建时间。
- `raw_data.fee_limit` - 执行智能合约交易时允许的最大能源成本。只有部署和触发智能合约交易需要设置，其他交易不需要。
- `signature` - 交易发送者的签名。这证明该交易只能来自发送者，而不是欺诈发送的。

## 交易类型
波场上有很多不同类型的交易，例如 TRX 转账交易、TRC10 转账交易、部署智能合约交易、触发智能合约交易、质押 TRX 交易等等。

要创建不同类型的交易，需要调用不同的 API。
例如，智能合约部署交易类型为 `CreateSmartContract`，需要调用钱包 `/deploycontract` API 来创建交易，
而质押 TRX 交易类型为 `FreezeBalanceV2Contract`，需要调用钱包 `/freezebalancev2` API 来创建交易。

```javascript
$ curl -X POST https://api.shasta.trongrid.io/wallet/freezebalancev2 -d '{"owner_address":"TCrkRWJuHP4VgQF3xwLNBAjVVXvxRRGpbA","frozen_balance": 2100000,"resource" : "BANDWIDTH","visible":true}' | jq
{
  "visible": true,
  "txID": "e54bab34838a59e85d5684e46a2e8e512cd11dfb07b35a9728adeaf3d2666fa6",
  "raw_data": {
    "contract": [
      {
        "parameter": {
          "value": {
            "frozen_balance": 2100000,
            "owner_address": "TCrkRWJuHP4VgQF3xwLNBAjVVXvxRRGpbA"
          },
          "type_url": "type.googleapis.com/protocol.FreezeBalanceV2Contract"
        },
        "type": "FreezeBalanceV2Contract"
      }
    ],
    "ref_block_bytes": "7139",
    "ref_block_hash": "d291dee525445093",
    "expiration": 1646902209000,
    "timestamp": 1646902151591
  },
  "raw_data_hex": "0a0271392208d291dee52544509340e8d39598f72f5a58080b12540a32747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e467265657a6542616c616e6365436f6e7472616374121e0a15411fafb1e96dfe4f609e2259bfaf8c77b60c535b9310a0968001180370a7939298f72f"
}
```

更多交易类型请参阅：[波场上的交易类型](https://github.com/tronprotocol/java-tron/blob/develop/protocol/src/main/protos/core/Tron.proto#L339)，更多 HTTP API 请参阅：`HTTP API`

## 交易生命周期
交易在其生命周期中会经历以下阶段：

- 交易的创建和签名。
- 交易被广播到波场网络，通过节点（包括区块产生节点）的验证和执行后，将被纳入交易缓存池。
- 出块节点按照放入的顺序从交易缓存池中一笔一笔取出交易，打包成新的区块，然后将新的区块广播到波场网络。
- 交易将被 "`confirmed`"。交易是否被确认取决于交易所在的区块是否被确认。波场的区块确认机制是，一个区块产生后，19 个不同的超级节点根据该区块产生后续区块，然后该区块被确认。

### 创建交易
有多种库和工具可用于创建交易。下面以 tronweb 创建 TRX 转账交易为例，说明如何创建交易：

```javascript
const unsignedTxn = await tronWeb.transactionBuilder.sendTrx("TVDGpn4hCSzJ5nkHPLetk8KQBtwaTppnkr", 100, "TNPeeaaFB7K9cmo4uQpcU32zGK8G1NYqeL");
 >{
    "visible": false,
    "txID": "9f62a65d0616c749643c4e2620b7877efd0f04dd5b2b4cd14004570d39858d7e",
    "raw_data": {
        "contract": [
            {
                "parameter": {
                    "value": {
                        "amount": 100,
                        "owner_address": "418840e6c55b9ada326d211d818c34a994aeced808",
                        "to_address": "41d3136787e667d1e055d2cd5db4b5f6c880563049"
                    },
                    "type_url": "type.googleapis.com/protocol.TransferContract"
                },
                "type": "TransferContract"
            }
        ],
        "ref_block_bytes": "0add",
        "ref_block_hash": "6c2763abadf9ed29",
        "expiration": 1581308685000,
        "timestamp": 1581308626092
    },
    "raw_data_hex": "0a020add22086c2763abadf9ed2940c8d5deea822e5a65080112610a2d747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5472616e73666572436f6e747261637412300a15418840e6c55b9ada326d211d818c34a994aeced808121541d3136787e667d1e055d2cd5db4b5f6c880563049186470ac89dbea822e"
}
```

### 签名交易
交易在发送之前需要使用发送者的私钥进行签名。

**交易签名生成流程**

1. 计算交易的哈希值。
2. 使用发送者的私钥对交易哈希进行签名。
3. 将生成的签名添加到交易实例中。

大多数 SDK 都实现了上述交易签名生成过程，并将其封装成一个接口供开发人员调用。以 tronweb 为例，用户可以直接调用 sign 方法来完成交易签名。

**使用 Tronweb 签名的示例**

使用 tronweb 对上面创建的交易进行签名：
    
```javascript
const signedTxn = await tronWeb.trx.sign(unsignedTxn, privateKey);
>{
    "visible": false,
    "txID":"9f62a65d0616c749643c4e2620b7877efd0f04dd5b2b4cd14004570d39858d7e",
    "raw_data":
    {
        "contract": [{<-->}],
        "ref_block_bytes": "0add",
        "ref_block_hash": "6c2763abadf9ed29",
        "expiration": 1581308685000,
        "timestamp": 1581308626092 
    },
    "raw_data_hex": "0a020add22086c2763abadf9ed2940c8d5deea822e5a65080112610a2d747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5472616e73666572436f6e747261637412300a15418840e6c55b9ada326d211d818c34a994aeced808121541d3136787e667d1e055d2cd5db4b5f6c880563049186470ac89dbea822e",
    "signature": [ "47b1f77b3e30cfbbfa41d795dd34475865240617dd1c5a7bad526f5fd89e52cd057c80b665cc2431efab53520e2b1b92a0425033baee915df858ca1c588b0a1800" ] 
 }
```

### 广播交易
节点收到用户发送的交易后，会在本地尝试验证并执行该交易，并向其他节点广播有效交易，丢弃无效交易，这将有效防止无效交易在网络中广播。

使用 tronweb 广播已签名的交易：

```javascript
const receipt = await tronWeb.trx.sendRawTransaction(signedTxn);
>{ 
    "result": true,
    "transaction":
    { 
        "visible": false,
        "txID": "9f62a65d0616c749643c4e2620b7877efd0f04dd5b2b4cd14004570d39858d7e",
        "raw_data":
        {
            "contract": [{<-->}],
            "ref_block_bytes": "0add",
            "ref_block_hash": "6c2763abadf9ed29",
            "expiration": 1581308685000,
            "timestamp": 1581308626092 
        },
        "raw_data_hex": "0a020add22086c2763abadf9ed2940c8d5deea822e5a65080112610a2d747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5472616e73666572436f6e747261637412300a15418840e6c55b9ada326d211d818c34a994aeced808121541d3136787e667d1e055d2cd5db4b5f6c880563049186470ac89dbea822e",
        "signature": [ "47b1f77b3e30cfbbfa41d795dd34475865240617dd1c5a7bad526f5fd89e52cd057c80b665cc2431efab53520e2b1b92a0425033baee915df858ca1c588b0a1800" ] 
    } 
 }
```

### 交易确认
交易是否得到确认取决于交易所在的区块是否得到确认。
波场的区块确认机制是，一个区块产生后，19 个不同的超级节点根据该区块产生后续区块，然后该区块被确认。

java-tron 节点提供了 `/walletsolidty/*` API，方便用户查询已确认的交易。
`/walletsolidty/*` 和 `/wallet/*` 的区别在于，用 `/wallet/*` 查询到的交易表示它已经上链，
但不一定是已确认的。而通过 `/walletsolidty/*` 查询到的交易，则表示该交易已在链上并已固化，也就是说，该交易已被确认。

对于不同的交易，有不同的方法来确定是否确认：

- **系统合约交易**：除创建智能合约类型和触发智能合约类型之外的所有交易类型均为系统合约交易。系统合约交易确认方法：
    * 只要能通过 `/walletsolidity/gettransactioninfobyid` 或 `/walletsolidity/gettransactionbyid` API 查询到交易，该交易就已确认。
- **智能合约交易**：包括创建智能合约和触发智能合约交易。由于需要在 TRON 虚拟机中执行，执行过程中可能会出现一些异常。这些交易虽然在链上，但并不意味着交易已成功执行。判断智能合约交易是否成功执行有两种方法：
    * 通过调用 `/walletsolidity/gettransactioninfobyid` API 查找 `transactionInfo.receipt.result` 是否等于 `success`。
    * 通过调用 `/walletsolidity/gettransactionbyid` API 查找 `transaction.ret.contractRet` 是否等于 `success`。
- **内部交易**：内部交易是指将代币转移到其他外部地址或合约中的合约地址的交易。首先，可以通过 `/walletsolidity/gettransactioninfobyid` API 查询内部交易，内部交易中的拒绝字段用于判断内部交易是否确认，但 HTTP 和 GRPC API 有所不同：
    * HTTP API：对于成功的交易，默认情况下不返回 rejected 字段。对于失败的交易，rejected 等于 `true`。
    * GRPC API：对于成功的交易，rejected 等于 `false`，表示当前 `internalTransaction` 未被丢弃；对于失败的交易，rejected 等于 `true`。


## 交易费用
除了查询操作，任何链上事务都会消耗[系统资源](./resource-model/index.md)。所有类型的交易都需要消耗带宽。
除了消耗带宽，智能合约部署和调用交易也会消耗能量。
当账户中的可用带宽或能源不足时，就需要燃烧 TRX 来支付相应的资源费用。
除了资源费，一些特殊交易还需要额外费用。

### 带宽费用
交易消耗的带宽量等于链上交易占用的字节数，包括三部分：交易的raw_data、交易签名、交易结果。这三部分经过 protobuf 序列化编码后占用的字节数就是交易消耗的带宽量。

当通过质押获得的带宽和账户中的每日免费带宽都不足时，就需要燃烧 TRX 来支付带宽费用：

!!!note ""
    支付带宽所燃烧的 TRX = 交易消耗的总带宽 * 带宽单价

目前带宽单价为 `1000sun`。

当 TRX 和 TRC10 转移交易的接收地址为未激活地址时，交易将激活接收地址。在这种情况下，如果调用者地址没有足够的带宽，交易将消耗 `0.1TRX` 作为带宽费。

有关如何估算交易的带宽消耗，请参阅此处。

### 能量费用
除了消耗带宽，智能合约的部署和调用事务也会消耗能量。
合约执行时，能量会根据指令逐一计算和扣除。
首先消耗的是通过质押获得的能量。如果这部分能量不够，账户的 TRX 将继续燃烧，以支付交易所需的能量费。

!!!note ""
    支付能量所燃烧的 TRX =（交易消耗的总能量 - 调用者账户中可用的能量）* 能量单价

目前的能量单价为 `210sun`。有关如何估算能量消耗，请参阅此处。

### 其他费用
对于某些特殊交易，除资源费外，还需支付额外费用。
如果交易发起人在交易中添加备注，则需要支付 1TRX 的额外交易备注费。
如果交易使用了多个签名，即交易中的签名数大于 1，则交易发起人需要额外支付 1TRX 的多签名费。
此外，以下特定类型的交易也需要支付交易费：



| 交易类型                            | 描述                                                 |    费用    |
|:--------------------------------|:---------------------------------------------------|:--------:|
| WitnessCreateContract           | 申请成为超级代表候选人                                        | 9999 TRX |
| AssetIssueContract              | 发行 TRC10 通证                                        | 1024 TRX |
| AccountCreateContract           | 创建新账户，即激活账户 |  1 TRX   |
| AccountPermissionUpdateContract | 更新账户权限                         | 100 TRX  |
| ExchangeCreateContract          | 创建交换对                             | 1024 TRX |

!!!note ""
    对于转账合约（`TransferContract`）或转账资产合约（`TransferAssetContract`）类型的交易，即 TRX 转账、TRC10 通证转账，如果目标地址未激活，则该交易还将触发新账户的创建，并扣除 1TRX 的新账户创建费。同时，如果交易发送者账户中通过质押获得的带宽不足，则需要支付 0.1TRX 作为带宽费。

## 内部交易
所谓交易一般是指外部账户触发的交易，例如智能合约调用交易。
但在智能合约交易执行过程中，合约可能会触发其他合约方法调用，或者将 `TRX/TRC10` 通证转移到外部账户，也可能会进行质押、投票、资源委托等操作。
智能合约的执行称为内部交易。因此内部交易是合约账户在 `TVM` 中触发的交易。

### 内部交易的生成
本文以去中心化交易所将 USDT 兑换为 TRX 为例，说明内部交易的产生。

以下是[TRX-USDT 交易对合约](https://tronscan.org/#/contract/TQn9Y2khEsLJW1ChVWFMSMeRDow5KcbLSE/code)的 `tokenToTrxSwapInput` 方法，它可以根据用户卖出的 USDT 数量为用户兑换 TRX：

```shell
function tokenToTrxSwapInput(uint256 tokens_sold, uint256 min_trx, uint256 deadline) public returns (uint256) {
    return tokenToTrxInput(tokens_sold, min_trx, deadline, msg.sender, msg.sender);
  }
  
function tokenToTrxInput(uint256 tokens_sold, uint256 min_trx, uint256 deadline, address buyer, address payable recipient) private nonReentrant returns (uint256) {
    require(deadline >= block.timestamp && tokens_sold > 0 && min_trx > 0);
    uint256 token_reserve = token.balanceOf(address(this));
    uint256 trx_bought = getInputPrice(tokens_sold, token_reserve, address(this).balance);
    uint256 wei_bought = trx_bought;
    require(wei_bought >= min_trx);
    recipient.transfer(wei_bought);

    require(address(token).safeTransferFrom(buyer, address(this), tokens_sold));
    emit TrxPurchase(buyer, tokens_sold, wei_bought);
    emit Snapshot(buyer,address(this).balance,token.balanceOf(address(this)));

    return wei_bought;
  }
```

我们可以看到，该合约函数中有四个代码涉及到触发其他合约或者向外部账户转账，分别是：

- 第 7 行：`token.balanceOf(address(this))`，用于查询该合约的 USDT 余额
- 第 11 行：`receiver.transfer(wei_bought)`，用于将 TRX 转入调用账户
- 第 13 行：`address(token).safeTransferFrom(buyer, address(this), tokens_sold))`, 用于将 USDT 从调用账户转入本合约
- 第 15 行：`token.balanceOf(address(this))`，用于查询该合约的 USDT 余额

上述四个代码分别对应于通过 [TRONSCAN](https://tronscan.org/#/transaction/50e6dd05c37b8666cf4a689fe6c0d52053b76b53d8649b256e6b9dca8c9df098/internal-transactions) 或 [API](https://api.trongrid.io/wallet/gettransactioninfobyid?value=50e6dd05c37b8666cf4a689fe6c0d52053b76b53d8649b256e6b9dca8c9df098) 接口查询的内部交易。

### 用例
内部交易可为用户提供一些重要信息。以下是一些可以在 dApp 内部使用内部交易信息的用例：

- 失败交易通知 - 如果内部交易失败，则整个交易将失败。通过内部交易可以达到通知用户故障点准确位置的目的，有助于快速定位和解决问题
- 智能合约监控 - 部署的智能合约可以通过内部交易与其他合约进行交互。要了解它何时以及与哪些合约进行交互，可以监控您的智能合约地址是否有任何内部交易。
- 智能合约分析 - 由于内部交易可能很复杂，因此深入了解很有帮助。通过查看智能合约执行的内部交易数量，可以了解该合约的性能。
- 批量交易 - 如果要向不同的发件人地址发送一批交易，可以使用内部交易来更方便、更安全地确保交易到达正确的地址。

### 保存内部交易
内部交易有很多用例，可以指导和告知用户交易的执行情况，但波场节点默认不保存内部交易信息，需要通过节点配置文件手动启用：

```shell
vm = {
    ...
  saveInternalTx = true
  saveFeaturedInternalTx = true
    ...
}
```

- saveInternalTx：是否保存内部交易
- saveFeaturedInternalTx：启用 `saveInternalTx` 时，是否保存与 Stake2.0 相关的内部交易

启用内部交易存储配置项后，重新启动节点。
从重启的那一刻起，节点将保存其内部交易。用户可以根据外部交易的交易 ID 查看其内部交易。
不支持直接通过内部交易的哈希值查看内部交易。API 是 `gettransactioninfobyid`。

### 内部交易示例
节点保存的内部交易包含以下信息：

- `hash`：内部交易的哈希值
- `caller_address`： 调用者地址
- `transferTo_address`：调用合约地址或接收 TRX/TRC10 通证的账户地址
- `callValueInfo.callValue`：转账的 TRX/TRC10 通证数量
- `callValueInfo.tokenId`：转账的 TRC10 名称或 id；转账 TRX 时，该字段为空。
- `note`：指令类型，如 `call`、`create`、`suicide`、`freezeBalanceV2ForEnergy`、`freezeBalanceV2ForBandwidth`、`unfreezeBalanceV2ForBandwidth` 等。
- `rejected`：内部交易是否执行失败，`true` 表示执行失败。
- `extra`：目前主要用于保存投票信息，以 JSON 格式记录投票 SR 及其得票数

根据不同的例子来看看智能合约内部交易中包含的信息：

#### 1. 调用智能合约方法
下面是例子，交易是外部地址调用 "`TQn9Y2khEsLJW1ChVWFMSMeRDow5KcbLSE`" 合约，
在 "`TQn9Y2khEsLJW1ChVWFMSMeRDow5KcbLSE`" 合约中，它调用 "`TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t`" 合约：

```javascript
{
    "id": "50e6dd05c37b8666cf4a689fe6c0d52053b76b53d8649b256e6b9dca8c9df098",
    ......
    "internal_transactions": [
        {
            "hash": "380f4d87271b83afcf5e867271ee2d30b36c19d3eeb15a043477bce7fd5b2079",
            "caller_address": "TQn9Y2khEsLJW1ChVWFMSMeRDow5KcbLSE",
            "transferTo_address": "TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t",
            "callValueInfo": [
                {}
            ],
            "note": "63616c6c"
        },
        .......
    ]
}
```

上述内部交易中包含的信息如下：

- `internal_transactions.caller_address` 是调用者地址，即外部地址直接调用的合约地址。
- `internal_transactions.transferTo_address` 是合约中调用的另一个合约的地址。
- `internal_transactions.callValueInfo` 在本例中，合约在调用其他合约时没有使用值传递 TRX/TRC10 通证，因此该字段值为空。如果在合约调用其他合约功能时附加了值信息，则会通过此字段反映出来。
- `internal_transactions.note` 是指令说明，采用十六进制格式，将其转换为字符串后，可以获得纯文本操作信息，本例中为 `call`

#### 2. TRX 转账
下面是例子，交易是外部地址调用 "`TQn9Y2khEsLJW1ChVWFMSMeRDow5KcbLSE`" 合约，在 "`TQn9Y2khEsLJW1ChVWFMSMeRDow5KcbLSE`" 合约中，它将 TRX 转账到 "`TQnpnLZJYMzH5xku535rAiYTnqYXTDTEHQ`" 地址：

```javascript
{
    "id": "50e6dd05c37b8666cf4a689fe6c0d52053b76b53d8649b256e6b9dca8c9df098",
     ......
    "internal_transactions": [
        ......
        {
            "hash": "f47fede2a45e722e6406421d0df16142e159ae7404525de5a595f4fc0c357e26",
            "caller_address": "TQn9Y2khEsLJW1ChVWFMSMeRDow5KcbLSE",
            "transferTo_address": "TQnpnLZJYMzH5xku535rAiYTnqYXTDTEHQ",
            "callValueInfo": [
                {
                    "callValue": 4514968563
                }
            ],
            "note": "63616c6c"
        },
        ......
    ]
}
```

- `Internal_transactions.caller_address` 是调用者的地址，即外部地址直接调用的合约的地址
- `Internal_transactions.transferTo_address` 是转账 TRX 的目标地址
- `Internal_transactions.callValueInfo[0].callValue` 是 TRX 转账数量，单位为 sun
- `internal_transactions.note` 是指令说明，采用十六进制格式，将其转换为字符串后，可以获得纯文本操作信息，本例中为 `call`

#### 3. TRC10 通证转账
TRC10 转账与 TRX 转账基本相同，除了以下两个字段：

- `Internal_transactions.callValueInfo[0].callValue` 是转账的 TRC10 转账数量
- `Internal_transactions.callValueInfo[0].tokenId` 是 TRC10 名称或 ID。由于[14号委员会提案](https://tronscan.org/#/proposal/14)允许相同的代币名称，因此在该提案生效之前（5537806之前的区块），该字段表示 TRC10 通证名称，该提案生效之后（5537806及之后的区块），该字段表示 TRC10 通证 ID。


#### 4. 质押 TRX
下面是例子，交易是外部地址调用 "`TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt`" 合约，在 "`TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt`" 中，合约质押 `1000TRX` 获取能量：

```javascript
{
    "id": "9d25a4fe417e0c7540cc5c5841e1d8c9215aec556d9b06e18910ed8b5088f0d8",
    ......
    "internal_transactions": [
        {
            "hash": "a3a4d666e7bf0729bbd8b5e5ad7afb7f8dd20191e7298ea9dbd17af345c96ed5",
            "caller_address": "TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt",
            "transferTo_address": "TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt",
            "callValueInfo": [
                {
                    "callValue": 1000000000
                }
            ],
            "note": "667265657a6542616c616e63655632466f72456e65726779"
        }
    ]
}
```

- `Internal_transactions.caller_address` 是质押发起者的地址，即外部地址直接调用的合约地址
- `Internal_transactions.transferTo_address` 是资源接收地址，即质押发起者地址，即外部地址直接调用的合约地址
- `Internal_transactions.callValueInfo[0].callValue` 是质押的 TRX 数量（以 sun 为单位）
- `Internal_transactions.note` 是指令说明，采用十六进制格式，将其转换为字符串后，可以获得纯文本操作信息，本例中为 `freezeBalanceV2ForEnergy`，表示合约质押 TRX 来获取能量。如果合约质押 TRX 来获取带宽，则该字段值为 `freezeBalanceV2ForBandwidth`

#### 5. 解押 TRX
下面是例子，交易是外部地址调用 "`TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt`" 合约，"`TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt`" 合约执行解押操作，将质押的 100TRX 解押以获得带宽：

```javascript
{
    "id": "dc110091fbd1568f8b264f287c7e0896d1afaf47b906a9e684fd17d57c7a1151",
    ......
    "internal_transactions": [
        {
            "hash": "16f73bdad5e9f984e082909b1028fff0b9865952131e681ca887446f8ec89918",
            "caller_address": "TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt",
            "transferTo_address": "TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt",
            "callValueInfo": [
                {
                    "callValue": 100000000
                }
            ],
            "note": "756e667265657a6542616c616e63655632466f7242616e647769647468"
        }
    ]
}
```

- `Internal_transactions.caller_address` 是解押发起方地址，即外部地址直接调用的合约地址
- `Internal_transactions.transferTo_address` 是 TRX 接收地址，即外部地址直接调用的合约地址
- `Internal_transactions.callValueInfo[0].callValue` 是解押的 TRX 数量（以 sun 为单位）
- `Internal_transactions.note` 是指令描述，在本例中为 `unfreezeBalanceV2ForBandwidth`，这意味着合约解押为获取带宽而质押的 TRX。如果合约解押为获取能量而质押的 TRX，则该字段的值为 `unfreezeBalanceV2ForEnergy`


#### 6. 资源代理
下面是例子，交易是外部地址调用 "`TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt`" 合约，"`TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt`" 合约将 500000000sun 的能量代理给 "`TUznHJfHe6gdYY7gvWmf6bNZHuPHDZtowf`" 地址：

```javascript
{
    "id": "342daa21f8865786295c45bb80e2f257740091e4e1a3a546b90daa51bcbcbd18",
    ......
    "internal_transactions": [
        {
            "hash": "58382a79c3af68c472383580309a81a9322e7520a48b6463917ba9219ca32a7d",
            "caller_address": "TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt",
            "transferTo_address": "TUznHJfHe6gdYY7gvWmf6bNZHuPHDZtowf",
            "callValueInfo": [
                {
                    "callValue": 500000000
                }
            ],
            "note": "64656c65676174655265736f757263654f66456e65726779"
        }
    ]
}
```

- `Internal_transactions.caller_address` 是资源委托地址，即外部地址直接调用的合约地址
- `Internal_transactions.transferTo_address` 是代理地址
- `Internal_transactions.callValueInfo[0].callValue` 是代理数量（单位为 sun）
- `Internal_transactions.note` 是指令描述，本例中为 `delegateResourceOfEnergy`，表示代理能量。如果是代理带宽，则该字段值为 `delegateResourceOfBandwidth`


#### 7. 资源回收
下面是例子，交易是外部地址调用 "`TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt`" 合约，"`TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt`" 合约取消对 "`TUoHaVjx7n5xz8LwPRDckgFrDWhMhuSuJM`" 地址的 200000000sun 的能量代理：

```javascript
{
    "id": "aa3961ffb0781d8b66d5e22368e92708135dac9c81eac1e2adcaa8546d729bc8",
    ......
    "internal_transactions": [
        {
            "hash": "1a8524704098770d9c6535e1112d1fb91855363c8366c93810f3cb56e8ee12bf",
            "caller_address": "TU8MbhYhurKv4T3xAHQKZCeP4DtFCmWLMt",
            "transferTo_address": "TUznHJfHe6gdYY7gvWmf6bNZHuPHDZtowf",
            "callValueInfo": [
                {
                    "callValue": 200000000
                }
            ],
            "note": "756e44656c65676174655265736f757263654f66456e65726779"
        }
    ]
}
```

- `Internal_transactions.caller_address` 是资源委托地址，即外部地址直接调用的合约地址
- `Internal_transactions.transferTo_address` 是回收地址
- `Internal_transactions.callValueInfo[0].callValue` 是回收的 TRX 数量（单位为 sun）
- `Internal_transactions.note` 是指令描述，本例为 `unDelegateResourceOfEnergy`，表示回收能量。如果是回收带宽，该字段值 `为unDelegateResourceOfBandwidth`


#### 8. 投票
下面是例子，交易是外部地址调用 "`TNaDYZaXEpL1LY8Uk4LtGTwwQrGzXTwss9`" 合约，"`TNaDYZaXEpL1LY8Uk4LtGTwwQrGzXTwss9`" 合约为超级代表 "`TUoHa....`" 和 "`TUznH..`" 分别投票 200 和 400 票：

```javascript
{
    "id": "58506325f692eee0bd730d97a0086f8b0c50e8aa5392b9e4b0edd5fb0916a718",
    ......
    "internal_transactions": [
        {
            "hash": "792b26cb6fbd1c92030721c62f7fd14522a94f412e04b05442d78e1ce743c9f4",
            "caller_address": "TNaDYZaXEpL1LY8Uk4LtGTwwQrGzXTwss9",
            "callValueInfo": [
                {}
            ],
            "note": "766f74655769746e657373",
            "extra": "{\"votes\":[{\"vote_address\":\"TUoHaVjx7n5xz8LwPRDckgFrDWhMhuSuJM\",\"vote_count\":200},{\"vote_address\":\"TUznHJfHe6gdYY7gvWmf6bNZHuPHDZtowf\",\"vote_count\":400}]}"
        }
    ],
    ......
}
```

- `Internal_transactions.caller_address` 是超级代表投票的地址，即外部地址直接调用的合约地址
- `Internal_transactions.transferTo_address` 是对于合约投票交易，该字段值为空，因此返回结果中不显示该字段
- `Internal_transactions.callValueInfo` 是对于合约投票交易，该字段值为空数组
- `Internal_transactions.note` 是指令描述，本例中是 `voteWitness`，表示超级代表投票操作
- `Internal_transactions.extra` 是投票详情，以 JSON 格式记录投票超级代表及其票数：`votes[i].vote_address` 为超级代表地址，`votes[i].vote_count` 为票数


#### 9. 提取奖励
下面是例子，交易是外部地址调用 "`TNaDYZaXEpL1LY8Uk4LtGTwwQrGzXTwss9`" 合约，"`TNaDYZaXEpL1LY8Uk4LtGTwwQrGzXTwss9`" 合约提取了 443803418sun 的奖励：

```javascript
{
    "id": "8dc24b5ce1399b553cd173529e77b22172d9abf31e9f50dd32c473bcc5234b71",
    ......
    "internal_transactions": [
        {
            "hash": "e5b047a4a3d64407c93a9e667a083fe52c52b24e859e3a44488249436e79c8d4",
            "caller_address": "TNaDYZaXEpL1LY8Uk4LtGTwwQrGzXTwss9",
            "transferTo_address": "TNaDYZaXEpL1LY8Uk4LtGTwwQrGzXTwss9",
            "callValueInfo": [
                {
                    "callValue": 443803418
                }
            ],
            "note": "7769746864726177526577617264"
        }
    ]
}
```

- `Internal_transactions.caller_address` 是提取奖励的地址，即外部地址直接调用的合约地址
- `Internal_transactions.transferTo_address` 是接收奖励的地址，即外部地址直接调用的合约地址
- `Internal_transactions.callValueInfo[0].callValue` 是提取的 TRX 奖励数量，单位为 sun
- `Internal_transactions.note` 是指令描述，本例为 `withdrawReward`，表示提取奖励