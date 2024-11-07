# 事件日志
***
事件日志是波场虚拟机最重要的功能之一，
用于在 TVM 运行合约时输出特定的二进制数据并记录在 TransactionInfo 中。
事件日志可以帮助开发者确认、检查和快速检索智能合约的特定状态。
本文将介绍事件机制的基础知识以及如何解码事件日志。

## 如何在合约中定义和触发事件
在 Solidity contract 中，可以用 `event` 关键字定义一个事件，
用 `emit` 关键字触发一个事件。定义事件时，不仅可以指定事件名称，
还可以指定多个参数来输出特定数据。以 TRC20 合约的转账事件为例：

```solidity
contract ExampleContractWithEvent {
    event Transfer(address indexed from，address indexed to, uint256 value);
    constructor() payable public{}
    function contractTransfer(address toAddress, uint256 amount){
        toAddress.transfer(amount);
        emit Transfer(msg.sender，toAddress, amount);
    }
}
```

- `event Transfer(address indexed from，address indexed to, uint256 value)`定义了一个包含三个参数的 Transfer 事件，第一个参数是 from，表示发送方地址；第二个参数是 to，表示接收方地址；第三个参数是值，表示转账数量。
- `emit Transfer(msg.sender,toAddress, amount)` 指定在转账完成后触发相应的事件。事件包含发送方地址、接收方地址和数量。

!!!note
    Solidity 规范一般要求事件名称大写，以区别于相应的函数。例如，事件 Transfer 和函数 transfer。

## 日志
Solidity 使用 LOG 指令在 TransactionInfo 中记录事件信息。
事件信息位于 TransactionInfo 的日志字段中。下面使用通过 `gettransactioninfobyid` API 获取的 TransactionInfo 来说明事件的结构：

```javascript
{
    "id": "88c66d08f15b983183c7f7d23e3fafec0320bcc837d67957a8bda58d04ca53e1",
    
    ......
    
    "log": [
        {
            "address": "a614f803b6fd780986a42c78ec9c7f77e6ded13c",
            "topics": [
                "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
                "00000000000000000000000079309abcff2cf531070ca9222a1f72c4a5136874",
                "00000000000000000000000081b64b1c09d448d25c9eeb3ee3b8f3348a694c96"
            ],
            "data": "00000000000000000000000000000000000000000000000000000000b2d05e00"
        }
    ]
}
```

日志：事务中触发的事件列表。每个事件都包含以下三个部分：

- `address`：合约地址。 为了与 EVM 兼容，TVM 中的地址是不带前缀 0x41 的十六进制地址，因此如果要解析日志中的地址，需要在日志地址的开头加上 41，然后将其转换为 Base58 格式。
- `topics`：事件的主题，包括事件本身和标记为索引的参数。使用主题保存索引参数的原因是，区块链使用 LevelDB 或 RockDB 等键值存储引擎。这些引擎通常支持前缀扫描操作。因此，将索引参数放入主题可以快速检索传输事件和具有特定 toAddress 的传输事件。
- `data`：事件的非索引参数，如数量。


## 日志解码
对于上述章节中的事件，如果要对其进行解码，首先必须知道事件的 ABI。以下是上述转账事件的 ABI：

```javascript
{
    "anonymous":false,
    "inputs":
    [
        {"indexed":true,"name":"from","type":"address"},
        {"indexed":true,"name":"to","type":"address"},
        {"indexed":false,"name":"value","type":"uint256"}
    ],
    "name":"Transfer",
    "type":"event"
}
```

沿 ABI 检查事件日志以解码数据：

- `topics[0]`：ddf252ad1be2c89b69c2b068fc378da952ba7f163c4a11628f55a4df523b3ef 是事件本身，值是 `keccak256('Transfer(address,address,uint256)')` 的计算结果，因此该事件是一个传输事件。事件的 keccak256 哈希值可通过 tronweb.sha3() 计算。

    !!!note
        keccak256 的参数是一个不含空格的字符串，否则计算出的哈希值将不同。

- `topics[1]`：0000000000000000000079309abcff2cf531070ca9222a1f72c4a5136874 是第一个索引参数。这里的地址是去掉前缀 0x41 后的 20 字节地址，因此对于该参数，只需获取最后 40 位数据，然后在前面加上 41 即可得到波场 HEX 格式的地址。
- `topics[2]`：000000000000000081b641c09d448d25c9eeb3ee3b8f3348a694c96 是第二个索引参数，即接收方账户地址，解析方法同上。
- `data`：0000000000000000000000000000000000000000b2d05e00 是一个非索引参数的值。如果有多个非索引参数，则根据 ABI 编码规则依次列出。详情请参阅 ABI 编码规则。在本例中，只有一个非索引参数，即转账数量。将十六进制数据转换为十进制即可。






