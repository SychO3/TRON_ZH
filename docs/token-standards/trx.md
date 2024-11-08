# TRX
***
TRX 是波场网络中最重要的加密货币，具有广泛的应用场景。
波场网络上的奖励以 TRX 形式发放。
用户可以通过质押 TRX 获取资源和投票权。TRX 也用作在 DeFi 贷款市场中的主要抵押品形式，以及在 NFT 市场中的计价单位等。

波场网络允许开发者创建去中心化应用程序（也称为 DAPP），
这些应用程序共享有限的波场网络资源，因此波场需要一种机制来防止 DAPP 意外或恶意占用所有网络资源。

波场网络资源的消耗通过带宽和能量来衡量，
其中带宽是衡量存储在区块链数据库中的交易大小的单位，以字节为单位，
而能量是衡量波场虚拟机执行特定操作所需计算能力的单位。
当用户进行交易时，他们必须支付执行交易所需的带宽和能量费用，波场支持通过燃烧 TRX 来支付带宽和能量。

因此，即便一个恶意的 DAPP 提交了无限循环，交易最终也会因为 TRX 的耗尽而终止，使网络恢复正常。

## 铸造 TRX

铸造是波场网络上创建新的 TRX 的过程。底层的波场协议创建新 TRX，用户无法自行创建 TRX。

当一个超级代表在波场网络上生成一个区块时，TRX 被铸造。
目前，对于每个新生成的区块，波场协议将产生 16TRX 的区块奖励和 160TRX 的投票奖励。
区块奖励和投票奖励是波场网络的动态参数，可以通过委员会提案进行修改。

## 燃烧 TRX

TRX 可以通过一种称为 “`燃烧`” 的过程被销毁。当 TRX 被燃烧时，它将永远从流通中移除。

波场上的每笔交易都会消耗带宽或能量。
当用户的带宽或能量不足时，他们需要燃烧 TRX 来支付交易所需的资源。TRX 的燃烧不仅有助于减少 TRX 的通胀，
还可以防止意外或恶意交易占用所有波场网络资源。

## TRX 单位

由于波场上的许多交易较小，波场引入了一种货币计量单位 sun，用于参考较小的金额。许多应用的技术实现是基于 sun 计算的。TRX 和 sun 的转换公式如下：

!!! quote ""
    1 TRX = 1_000_000 sun

## 转账 TRX

转账 TRX 是一种 `TransferContract` 类型的波场网络交易，用于将 TRX 从一个账户转移到另一个账户。以下是使用 HTTP API 和 tronweb SDK 转账 TRX 的示例：

### HTTP API
以下是在全节点 HTTP 接口 `wallet/createtransaction` 中创建未签名的 TRX 转账交易的过程：

```shell
curl -X POST  https://api.shasta.trongrid.io/wallet/createtransaction -d 
    '{
        "to_address": "TVDGpn4hCSzJ5nkHPLetk8KQBtwaTppnkr", 
        "owner_address": "TM2TmqauSEiRf16CyFgzHV2BVxBejY9iyR", 
        "amount": 10000000,
        "visible":true
    }'
```

通过上述接口创建未签名交易后，您需要对交易进行签名并广播，以最终完成 TRX 转账。

### tronweb SDK
让我们通过 tronweb 创建一个 TRX 转账交易：

```javascript
const privateKey = "..."; 
var fromAddress = "TM2TmqauSEiRf16CyFgzHV2BVxBejY9iyR"; //address _from
var toAddress = "TVDGpn4hCSzJ5nkHPLetk8KQBtwaTppnkr"; //address _to
var amount = 10000000; //amount，in sun
// Create an unsigned TRX transfer transaction
const tradeobj = await tronWeb.transactionBuilder.sendTrx(
      toAddress,
      amount,
      fromAddress
);
// Sign
const signedtxn = await tronWeb.trx.sign(
      tradeobj,
      privateKey
);
// Broadcast
const receipt = await tronWeb.trx.sendRawTransaction(
      signedtxn
).then(output => {
  console.log('- Output:', output, '\n');
  return output;
});
```

## 查询 TRX 余额
### HTTP API
您可以通过完整节点的 HTTP API 接口 `wallet/getaccount` 查询账户的 TRX 余额。返回值中的 `balance` 字段表示 TRX 余额，单位是 sun：

```shell
curl -X POST  https://api.shasta.trongrid.io/wallet/getaccount -d 
      '{"address": "TM2TmqauSEiRf16CyFgzHV2BVxBejY9iyR",
        "visible": true
        }'
```

### tronweb SDK
让我们通过 tronweb 查询一个账户的 TRX 余额：

```javascript
const privateKey = "..."; 
var address = "TM2TmqauSEiRf16CyFgzHV2BVxBejY9iyR"; 
// Query the information of an account, and get the balance through the 'balance' in the return value.
var tradeobj = await tronWeb.trx.getAccount(
      address,
).then(output => {console.log('- Output:', output, '\n');});
```