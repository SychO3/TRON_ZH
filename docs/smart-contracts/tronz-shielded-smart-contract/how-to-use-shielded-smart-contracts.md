# 如何使用隐私智能合约
***
## z-地址生成

z-地址，即隐私地址，是TRONZ使用的地址格式。该格式由一个`sk`和一个`d`密钥决定。用户可以发布其z-地址并接收隐私交易。

```plaintext
// sk: spending key => ask, nsk, ovk
// ask: spend authorizing key, 256 => ak
// nsk: proof authorizing key, 256 => nk
// ovk: outgoing viewing key, 256
// ivk: incoming viewing key, 256 => pkD
// d: diversifier, 11bytes
// pkD: the public key of the address, g_d^ivk
// pkD + d => z-addr
```

`sk`是隐藏的私钥。`d`是从`sk`生成的不同地址的标识符，可以用于实现HD钱包。所有其他密钥在不同场合使用。

以一个地址为例：`ztron1m445gx74mjuuyhkyru5hrx886jszfga4a7dk3mg4uarrl0cru649jz4928tm6rqul2pg645hqv5`。
该地址为[bech32](https://en.bitcoin.it/wiki/Bech32)格式。`ztron1`是固定前缀，其余部分为`pkD`和`d`的编码值。

相比于原始的TRON地址格式`T-....`，可以称其为透明地址或T-地址。

与密钥和z-地址相关的API有：

```plaintext
wallet/getspendingkey
> 生成sk

wallet/getexpandedspendingkey
> sk => aks, nsk, ovk

wallet/getakfromask
> ask => ak

wallet/getnkfromnsk
> nsk => nk

wallet/getincomingviewingkey
> ak, nk => ivk

wallet/getdiversifier
> 生成d

wallet/getzenpaymentaddress
> ivk, d => z-addr, pkD

wallet/getnewshieldedaddress
> 一次性生成上述所有密钥和地址
```

与隐私交易相关的3个主要任务是：

- 从T-地址转账到z-地址，为隐私TRC20`铸币`
- 在z-地址之间转账，为隐私TRC20`转账`
- 从z-地址转账到T-地址，为隐私TRC20`燃烧`

## 隐私TRC20合约

```plaintext
constructor (address trc20ContractAddress, uint256 scalingFactorExp)

function burn(bytes32[10] input, bytes32[2] spendAuthoritySignature, uint256 rawValue, bytes32[2] bindingSignature, address payTo, bytes32[3] c)
    => burn(bytes32[10],bytes32[2],uint256,bytes32[2],address,bytes32[3]) [4d013fde]

function mint(uint256 rawValue, bytes32[9] output, bytes32[2] bindingSignature, bytes32[21] c)
    => mint(uint256,bytes32[9],bytes32[2],bytes32[21]) [855d175e]

function transfer(bytes32[10][] input, bytes32[2][] spendAuthoritySignature, bytes32[9][] output, bytes32[2] bindingSignature, bytes32[21][] c)
    => transfer(bytes32[10][],bytes32[2][],bytes32[9][],bytes32[2],bytes32[21][]) [9110a55b]

function scalingFactor() view returns (uint256)
    => scalingFactor() [ed3437f8]

function getPath(uint256 position) view returns (bytes32, bytes32[32])
    => getPath(uint256) [e1765073]
```

## 使用说明

### **前提条件 - 一个隐私TRC20合约**

隐私合约在 [ShieldedTRC20.sol](https://github.com/Federico2014/java-tron/blob/feature/shieldedUSDT/deploy/ShieldedTRC20.sol)。

只能使用来自波场的 [Solidity 变体 ](https://github.com/tronprotocol/solidity)进行编译，使用的代码分支是 `develop`。

使用构造函数参数 `(TF17BgPaZYbz8oxbjhriubPDsA7ArKoLX3, 18)` 部署代码。我们得到一个隐私TRC20合约地址 `TNnFMMykZzwhPZkurKtNMyVGvgeSkCrnPi`。

其中，`TF17BgPaZYbz8oxbjhriubPDsA7ArKoLX3` 是Nile测试网络中`JST通证`的TRC20地址，`1`是缩放因子的指数。

### **转账到 z-地址 - 铸币**

在任何转账前，使用原始TRC20的 `approve` 方法批准所需数量的通证给隐私TRC20合约。

假设我们想将通证转账到地址 `ztron1s2s9fpf2v2l3d8mgzf7dqnfptkrlmyekvaqlw50lpf8dz8xkdgphjuxaysh4h0wvml8qzjzrv36`。

首先，创建 `rcm`：

```shell
curl https://api.nileex.io/wallet/getrcm
{"value": "b9a22cedc38da726f056299e4d09bae107df201066647b0a58912f33d0f49b00"}
```

然后，构造隐私合约参数：

```python
{
  'from_amount': '200000000000000000000',
  'shielded_receives': {'note': {
      'value': 200,
      'payment_address': 'ztron1s2s9fpf2v2l3d8mgzf7dqnfptkrlmyekvaqlw50lpf8dz8xkdgphjuxaysh4h0wvml8qzjzrv36',
      'rcm': 'b9a22cedc38da726f056299e4d09bae107df201066647b0a58912f33d0f49b00',
      'memo': 'HEX'}},
  'shielded_TRC20_contract_address': '418C8705769E5EC53F5F9C42A3BC3305624AD37192'
}
```

- `from_amount` 是字符串类型，
- `value` 是 `from_amount` 除以缩放因子。

然后我们得到响应，唯一需要关注的是 `trigger_contract_input`：

```python
{
 'binding_signature': '......',
 'message_hash': '.....',
 'parameter_type': 'mint',
 'receive_description': [{'c_enc': '..........',
                          'c_out': '.....',
                          'epk': '...',
                          'note_commitment': '.....',
                          'value_commitment': '.....',
                          'zkproof': '.....'}],
 'trigger_contract_input': '.....'
}
```

将 `[855d175e]`(`mint() 的签名`) 与 `trigger_contract_input` 连接，我们得到调用参数。使用 T-地址签署 `TriggerSmartContract` 交易并广播。

这是 `TriggerSmartContract` 参数的示例：

```python
{
 'contract_address': '418c8705769e5ec53f5f9c42a3bc3305624ad37192',
 'fee_limit': 100000000,
 'function_selector': 'mint(uint256,bytes32[9],bytes32[2],bytes32[21])',
 'owner_address': '41458915d507cd3384f4e15e9bb2e5ebd0fa974061',
 'parameter': '00000000000000000000000000000000000000000000000ad78ebc5ac620000056ae06ce40ce9eb28fde5f76c80617b81a1f1669c0a8d04c09265bc4b2058f04606ef2f138dfe02b08343be5345c15e61e287541aee884dc6f9e1d3986b8a9a12564bc61bfad63c09ad9f39f782acb96027cbb89bd33a99d6db430fddba41393aca4cfdd915e87b6b89a1d54e310489b6c746ba4348cc0af31029298526d2fe7714cf070ea658ba3fa25888afa0f8f12b67950787ef3f7a8b96de905b19ca6562b86216e18ea53bcb38ab2c90a146e4fb41bda55dfee5aa46d886068c941ea10161861426afd6fd9718ac21da3c548ddf2b33e39cca3b2ee4965e891eae180c62d35014c02994e19e00e383577c578b7866030b896e90f04b05bfc8aa0aa06d468662aa97722d3ff6c78306568ceb7be2fbda34afb1bd2f33d4ff3266e2e6edc0b6048c8deecf69c9a9bf6702cce07b2ffc5f7ad2d18da03fc5588f4c984cbe9b08ce2973d0cb1ee8113242126caddf7e406d8bcc80c2d24e4dd88d0b5870b05646ca4ca540aa16d267831a24cd53c06dfb9764e1be035fabeb2e814c6d18d3fd1091cf8b05296b6084dffd6fe6d26046897174b1a0ca8fa370eba2d4dd1ac9532970f41da1af41d78bc54260820e9d01ae8bfd4af30250aeaa4b95b153fccd6771e47cfb49aea19bfc3f24356ae95f715b6700b7c4ec5856d43f1e7383efc8805e7c12da045b832d07b2634b0238c1a1321010bd73f0b5936882a31f51a75819648315c6b20a84500799e35d1831872ea916b2c1d95102432664a388b3b96e930783c98024bf3320029e982dca8f16b7690447ea744dc278243d38bf12f014fc1a8e27dff649f15bfcacd65f999c951cb8213b5cfd314ae2f136d3152a013fde43e8b620d3f47ed5146cb0e778aa17e33a5abae3b0d9fd5d63be78b2e73bf9668e5bc1a9761442bee34700f816c96796b96d6e755188fa7028fa281350a55ecb65333d4a65fb48e0d08dfbb2b78df40d9a2f0986587c2503db2e70d2b8e89ff5feec7b25543c8bb2bbfbb5cb994b97667e14c5e53510e6416cea932f7a705b24babd22dd1f786bd2e4c9675531ca77d6a244854059cf5cd9bd03a1a0866d767b33feb180e687642c6f84051476cda46b460751e19f7e38356445dd66beed81b83d2ccf9dfee4b5665581ba5ead308f44f89b01a844ebd3015cc797e32c9808b3854a4fd3c3d40e55e77b48d0e299a7b627f2bf944ae4025712afa5801a456a757a9f5039369ccaf311cdd2c5402784501d4363502153de3b742d043dfe5ddc1a6317172ed075c451a7d386d5b622b8661a8070d09a8843151b2e369fefff455068094566549b0e05d79f8fde9a0e9ca11a10fc7b03d37ce56a5c35b674df80ee66fb6932ed4db7f3c5f55d355765a99e4db08c89e48280dff5b79fcc4c722dc19edcc118b9111a026b9b36deeec100b325d582b000000000000000000000000'
}
```

交易： [https://nile.tronscan.org/#/transaction/d05078358d359450aa8f57df7722d2d338e53af15362f852b1d7e6b5c4b8f47b](https://nile.tronscan.org/#/transaction/d05078358d359450aa8f57df7722d2d338e53af15362f852b1d7e6b5c4b8f47b)

### **查询接收的记录**

使用 `wallet/scanshieldedtrc20notesbyivk`，传入 `ivk`, `ak`, `nk`。一次最多扫描1000个区块。

返回的记录有一个字段叫 `is_spent`。

### **在 z-地址之间转账 - 转账**

所有转账基于记录。可以将1-2条记录转为1-2条记录。总余额必须匹配。

`alpha` 和 `rcm` 通过 `wallet/getrcm` API生成。

### **查询发出的记录**

使用 `wallet/scanshieldedtrc20notesbyovk`，传入 `ovk`。

### **记录是否已消费**

使用 `wallet/isshieldedtrc20contractNoteSpent`，传入 `ak`, `nk`。

### **从 z-地址转到 T-地址 - 燃烧**

几乎与转账相同。`TriggerSmartContract` 交易可以由任何人广播。

## **参考资料**

详细使用文档在[此处](https://github.com/tronprotocol/documentation-en/blob/master/docs/mechanism-algorithm/shielded-TRC20-contract.md)。