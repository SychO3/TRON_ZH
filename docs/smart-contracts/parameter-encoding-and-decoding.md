# 参数编码与解码
***
本文主要介绍在波场网络中触发智能合约时如何对参数进行编码和解码。参数的编码和解码遵循 Solidity ABI 编码规则。

## ABI 编码规范

本章节主要通过示例介绍 ABI 编码规则。关于详细的 ABI 编码规则，请参考 Solidity 文档中的 ABI Specification。

### 函数选择器

合约函数调用的数据字段的前四个字节是函数选择器，指定要调用的函数。

函数选择器是函数签名的 `Keccak-256 `哈希值的前（左，高位序）四个字节。函数签名仅包含函数名和参数类型，不包含参数名和空格。以 `transfer(address _to, uint256 _value)` 为例，其函数签名为 `transfer(address, uint256)`。

您可以通过调用 tronweb.sha3 接口获得函数签名的 `Keccak-256` 哈希值。

### 参数编码

从第五个字节开始，后续是编码的参数。这种编码在其他地方也使用，例如返回值和事件参数也以相同方式编码，不包括指定函数的四个字节。

#### 类型

我们区分静态类型和动态类型。静态类型就地编码，动态类型在当前块之后的单独分配位置编码。

静态类型：固定长度参数，例如 `uint256`、`bytes32`、`bool`（布尔类型为 uint8，只能为 0 或 1）。以 `uint256` 为例，对于类型为 `uint256` 的参数，即使值为 1，也需要填充 0 以达到 256 位，即 32 字节，因此静态参数的长度是固定的，与值无关。

动态类型：动态参数的长度不确定。动态参数类型包括：`bytes`、`string`、`任意 T 的 T[]`、`任意动态 T` 和`任意 k >= 0 的 T[k]`、如果 Ti 在某些 1 <= i <= k 中是动态的(T1,...,Tk)。

#### 静态参数编码

**示例 1**

```solidity
function baz(uint32 x, bool y) public pure returns (bool r) { r = x > 32 || y; }
```

函数签名为 `baz(uint32,bool)`，函数签名的 Keccak-256 值为 `0xcdcd77c0992ec5bbfc459984220f8c45084cc24d9b6efed1fae540db8de801d2`，其函数选择器为 `0xcdcd77c0`。

参数编码以十六进制表示，每两个十六进制数字占用一个字节。由于静态参数的最大长度是 256 位，在编码时，每个静态参数的长度为 256 位，即 32 字节，共 64 位十六进制数字。当参数小于 256 位时，左边用 0 填充。

将参数集 `(69, true)` 传递给 `baz` 方法，编码结果如下：

将十进制数 69 转换为十六进制 45，并在左侧添加 0 使其占用 32 字节，结果为：
```
0x0000000000000000000000000000000000000000000000000000000000000045
```

布尔值 `true` 是 `uint8` 的 1，其十六进制值也是 1，并在左侧添加 0 使其占用 32 字节，结果为：
```
0x0000000000000000000000000000000000000000000000000000000000000001
```

总共为：

```
0xcdcd77c000000000000000000000000000000000000000000000000000000000000000450000000000000000000000000000000000000000000000000000000000000001
```

**示例 2**

同样地，对于静态类型的 bytes 类型数据，在编码时需要填充到 32 字节。不同之处在于，bytes 类型数据需要在右侧填充。以下面的函数为例：

```solidity
function bar(bytes3[2] memory) public pure {}
```

函数签名为 `bar(bytes3[2])`，函数选择器为：`0xfce353f6`。

将参数集 `(abc, def)` 传递给该函数，编码结果如下：

字母 `a`、`b`、`c` 的 ASCII 值分别为十进制 97、98、99，十六进制为 61、62、63。如果参数小于 32 字节，则需要在右侧用 0 填充，结果为：
```
0x6162630000000000000000000000000000000000000000000000000000000000
```

字母 `d`、`e`、`f` 的 ASCII 值分别为十进制 100、101、102，十六进制为 64、65、66。如果参数小于 32 字节，则需要在右侧用 0 填充，结果为：
```
0x6465660000000000000000000000000000000000000000000000000000000000
```

总共为：

```
0xfce353f661626300000000000000000000000000000000000000000000000000000000006465660000000000000000000000000000000000000000000000000000000000
```

#### 动态参数编码

对于动态参数，由于其长度不确定，需要使用固定长度的偏移量来占据空间，记录动态参数实际位置的偏移字节数，然后对数据进行编码。

以下述函数为例 `f(uint,uint32[],bytes10,bytes)`，当传递参数 `(0x123, [0x456, 0x789], "1234567890", "Hello, world!")` 时，编码结果如下：

第一个静态参数编码：未标明长度的 uints 被视为 uint256，0x123 的编码结果为：

```
0x0000000000000000000000000000000000000000000000000000000000000123
```

第二个动态参数的偏移量：对于 `uint32[]`，由于数组长度未知，首先使用偏移量占位，偏移量记录该参数起始位置的字节数。在 `uint32` 参数的正式编码之前，有：第一个参数 uint 的编码（32 字节），第二个参数 `uint32[]` 的偏移量（32 字节），第三个参数 `bytes10` 的编码（32 字节），第四个参数 `bytes` 的偏移量（32 字节），因此值编码的起始字节数应为 128，即 0x80，编码结果为：

```
0x0000000000000000000000000000000000000000000000000000000000000080
```

第二个动态参数的值编码：将数组 `[0x456, 0x789]` 传递给 `uint32[]`。对于动态参数，首先记录其长度，即 0x2，然后对值进行编码。此参数的编码结果为：

```
0000000000000000000000000000000000000000000000000000000000000002
0000000000000000000000000000000000000000000000000000000000000456
0000000000000000000000000000000000000000000000000000000000000789
```

第三个静态参数编码："1234567890" 是静态 `bytes10` 参数，将其转换为十六进制格式并补 0，结果为：

```
0x3132333435363738393000000000000000000000000000000000000000000000
```

第四个动态参数的偏移量：此参数类型为 bytes，是动态类型，因此首先使用偏移量占位。参数实际内容之前的内容为：1. 第一个参数 uint 的编码（32 字节），2. 第二个参数 `uint32[]` 的偏移量（32 字节），3. 第三个参数 `bytes10` 的编码（32 字节），4. 第四个参数 `bytes` 的偏移量（32 字节），5. 第二个参数 `uint32[]` 的编码（96 字节）。因此偏移量应为 224，即 0xe0。

```
0x00000000000000000000000000000000000000000000000000000000000000e0
```

第四个动态参数的值编码：对于 `bytes` 类型的参数值："Hello, world!"，首先记录其长度 13，即 0xd。然后将字符串转换为十六进制字符，即：0x48656c6c6f2c20776f726c642100000000000000000000000000000000000000。此参数的编码结果为：

```
000000000000000000000000000000000000000000000000000000000000000d
48656c6c6f2c20776f726c642100000000000000000000000000000000000000
```

所有参数编码完成，最终数据为：

```
0x8be65246 - 函数选择器
0000000000000000000000000000000000000000000000000000000000000123 - 0x123 的编码
0000000000000000000000000000000000000000000000000000000000000080 - [0x456, 0x789] 的偏移量
3132333435363738393000000000000000000000000000000000000000000000 - "1234567890" 的编码
00000000000000000000000000000000000000000000000000000000000000e0 - "Hello, world!" 的偏移量
0000000000000000000000000000000000000000000000000000000000000002 - [0x456, 0x789] 的长度
0000000000000000000000000000000000000000000000000000000000000456 - 0x456 的编码
0000000000000000000000000000000000000000000000000000000000000789 - 0x789 的编码
000000000000000000000000000000000000000000000000000000000000000d - "Hello, world!" 的长度
48656c6c6f2c20776f726c642100000000000000000000000000000000000000 - "Hello, world!" 的编码
```

## 参数的编码与解码

在理解 ABI 编码规则之后，可以在代码中使用 ABI 规则进行参数的编码与解码。波场社区为开发者提供了许多 SDK 或库，一些 SDK 已经封装了参数的编码与解码，可以直接调用，例如 trident-java。下面将使用 trident-java SDK 和 JavaScript 库来说明如何在代码中进行参数的编码与解码。

### 参数编码

我们以 USDT 合约中的转账函数为例：

```solidity
function transfer(address to, uint256 value) public returns (bool);
```

假设您向地址 `412ed5dd8a98aea00ae32517742ea5289761b2710e` 转账 50000 USDT，并调用 `triggersmartcontract` 接口，如下所示：

```curl
curl -X POST https://127.0.0.1:8090/wallet/triggersmartcontract -d '{
"contract_address":"412dd04f7b26176aa130823bcc67449d1f451eb98f",
"owner_address":"411fafb1e96dfe4f609e2259bfaf8c77b60c535b93",
"function_selector":"transfer(address,uint256)",
"parameter":"0000000000000000000000002ed5dd8a98aea00ae32517742ea5289761b2710e0000000000000000000000000000000000000000000000000000000ba43b7400",
"call_value":0,
"fee_limit":1000000000,
"call_token_value":0,
"token_id":0
}'
```

在上述命令中，参数的编码需要符合 ABI 规则。

#### 使用 JavaScript 进行参数编码示例

对于 JavaScript 用户，可以使用 `ethers` 库，以下是示例代码：

```javascript
// 推荐使用 ethers 4.0.47 版本
var ethers = require('ethers')

const AbiCoder = ethers.utils.AbiCoder;
const ADDRESS_PREFIX_REGEX = /^(41)/;
const ADDRESS_PREFIX = "41";

async function encodeParams(inputs){
    let typesValues = inputs
    let parameters = ''

    if (typesValues.length == 0)
        return parameters
    const abiCoder = new AbiCoder();
    let types = [];
    const values = [];

    for (let i = 0; i < typesValues.length; i++) {
        let {type, value} = typesValues[i];
        if (type == 'address')
            value = value.replace(ADDRESS_PREFIX_REGEX, '0x');
        else if (type == 'address[]')
            value = value.map(v => toHex(v).replace(ADDRESS_PREFIX_REGEX, '0x'));
        types.push(type);
        values.push(value);
    }

    console.log(types, values)
    try {
        parameters = abiCoder.encode(types, values).replace(/^(0x)/, '');
    } catch (ex) {
        console.log(ex);
    }
    return parameters

}

async function main() {
    let inputs = [
        {type: 'address', value: "412ed5dd8a98aea00ae32517742ea5289761b2710e"},
        {type: 'uint256', value: 50000000000}
    ]
    let parameters = await encodeParams(inputs)
    console.log(parameters)
}

main()
```

输出：

```
0000000000000000000000002ed5dd8a98aea00ae32517742ea5289761b2710e0000000000000000000000000000000000000000000000000000000ba43b7400
```

#### 使用 trident-java 进行参数编码示例

参数编码过程已在 trident 中封装，只需选择参数类型并传入参数值。参数类型位于 `org.tron.trident.abi.datatypes` 包中，请根据参数类型选择合适的 Java 类。以下示例代码展示了如何使用 trident 生成合约的数据信息。主要步骤如下：

1. 构造一个 `Function` 对象，需三个参数：函数名、输入参数和输出参数。详情见 `Function` 代码。
2. 调用 `FunctionEncoder.encode` 函数对 `Function` 对象进行编码，并生成合约交易的数据。

```java
public void SendTrc20Transaction() {
    ApiWrapper client = ApiWrapper.ofNile("3333333333333333333333333333333333333333333333333333333333333333");

    org.tron.trident.core.contract.Contract contr  = client.getContract("");
    
    // transfer(address,uint256) returns (bool)
    Function trc20Transfer = new Function("transfer",
                                          Arrays.asList(new Address("TVjsyZ7fYF3qLF6BQgPmTEZy1xrNNyVAAA"),new Uint256(BigInteger.valueOf(10).multiply(BigInteger.valueOf(10).pow(18)))),
                                          Arrays.asList(new TypeReference<Bool>() {})
                                         );

    String encodedHex = FunctionEncoder.encode(trc20Transfer);

    TriggerSmartContract trigger =
            TriggerSmartContract.newBuilder()
                .setOwnerAddress(ApiWrapper.parseAddress("TJRabPrwbZy45sbavfcjinPJC18kjpRTv8"))
                .setContractAddress(ApiWrapper.parseAddress("TF17BgPaZYbz8oxbjhriubPDsA7ArKoLX3"))
                .setData(ApiWrapper.parseHex(encodedHex))
                .build();

    System.out.println("trigger:\n" + trigger);

    TransactionExtention txnExt = client.blockingStub.triggerContract(trigger);
    System.out.println("txn id => " + Hex.toHexString(txnExt.getTxid().toByteArray()));
}
```


### 参数解码

在上节的参数编码中，调用 `triggersmartcontract` 生成一个交易对象，然后对其进行签名和广播。交易成功上链后，可以通过 `gettransactionbyid` 获取链上的交易信息：

```curl
curl -X POST \
  https://api.trongrid.io/wallet/gettransactionbyid \
  -d '{"value" : "1472178f0845f0bfb15957059f3fe9c791e7e039f449c3d5a843aafbc8bbdeeb"}'
```

结果如下：

```json
{
    "ret": [
        {
            "contractRet": "SUCCESS"
        }
    ],
    ..........
    "raw_data": {
        "contract": [
            {
                "parameter": {
                    "value": {
                        "data": "a9059cbb0000000000000000000000002ed5dd8a98aea00ae32517742ea5289761b2710e0000000000000000000000000000000000000000000000000000000ba43b7400",
                        "owner_address": "418a4a39b0e62a091608e9631ffd19427d2d338dbd",
                        "contract_address": "41a614f803b6fd780986a42c78ec9c7f77e6ded13c"
                    },
                    "type_url": "type.googleapis.com/protocol.TriggerSmartContract"
                },
    ..........
}
```

返回值中的 `raw_data.contract[0].parameter.value.data` 字段是被调用的 `transfer(address to, uint256 value)` 函数及其参数。数据字段的前四个字节 `a9059cbb` 是函数选择器，来自于 `transfer(address, uint256)` 的 ASCII 格式经过 Keccak-256 操作后的前 4 个字节，用于虚拟机定位函数。后面的部分是参数，与参数编码章节中 `wallet/triggersmartcontract` 接口的参数相同。

函数选择器，即数据的前四个字节，通过 Keccak-256 获得，不可逆。函数签名可以通过两种方式获得：

1. 如果可以获得合约的 ABI，则可以计算每个合约函数的选择器，并与数据的前四个字节比较来判断函数。
2. 合约生成的合约在链上可能没有 ABI。合约部署者也可以通过 `clearAbi` 接口清除链上的 ABI。当无法获得 ABI 时，可以尝试通过以太坊签名数据库查询函数。

参数解码请参考以下内容。

#### 使用 JavaScript 进行参数解码示例

**解码数据**

以下 JavaScript 代码解码数据字段，并获取 `transfer` 函数传递的参数：

```javascript
var ethers = require('ethers')

const AbiCoder = ethers.utils.AbiCoder;
const ADDRESS_PREFIX_REGEX = /^(41)/;
const ADDRESS_PREFIX = "41";

// types: 参数类型列表，如果函数有多个返回值，列表中类型的顺序应与定义顺序一致
// output: 解码前的数据
// ignoreMethodHash：解码函数返回值时，填 false；解码 gettransactionbyid 结果中的数据字段时，填 true

async function decodeParams(types, output, ignoreMethodHash) {

    if (!output || typeof output === 'boolean') {
        ignoreMethodHash = output;
        output = types;
    }

    if (ignoreMethodHash && output.replace(/^0x/, '').length % 64 === 8)
        output = '0x' + output.replace(/^0x/, '').substring(8);

    const abiCoder = new AbiCoder();

    if (output.replace(/^0x/, '').length % 64)
        throw new Error('The encoded string is not valid. Its length must be a multiple of 64.');
    return abiCoder.decode(types, output).reduce((obj, arg, index) => {
        if (types[index] == 'address')
            arg = ADDRESS_PREFIX + arg.substr(2).toLowerCase();
        obj.push(arg);
        return obj;
    }, []);
}


async function main() {

    let data = '0xa9059cbb0000000000000000000000004f53238d40e1a3cb8752a2be81f053e266d9ecab000000000000000000000000000000000000000000000000000000024dba7580'

    let result = await decodeParams(['address', 'uint256'], data, true)
    console.log(result)
}

main()
```

示例代码输出：

```
[ '414f53238d40e1a3cb8752a2be81f053e266d9ecab', BigNumber { _hex: '0x024dba7580' } ]
```

**解码合约查询操作的返回值**

我们以 USDT 合约中的查询函数为例：

```solidity
balanceOf(address who) public constant returns (uint)
```

假设您查询地址 `410583A68A3BCD86C25AB1BEE482BAC04A216B0261` 的余额，并调用 `triggerconstantcontract` 接口，如下所示：

```curl
curl -X POST https://127.0.0.1:8090/wallet/triggerconstantcontract -d '{
"contract_address":"419E62BE7F4F103C36507CB2A753418791B1CDC182",
"function_selector":"balanceOf(address)",
"parameter":"000000000000000000000041977C20977F412C2A1AA4EF3D49FEE5EC4C31CDFB",
"owner_address":"41977C20977F412C2A1AA4EF3D49FEE5EC4C31CDFB"
}'
```

结果如下：

```json
{
    "result": {
        "result": true
    },
    "constant_result": [
        "000000000000000000000000000000000000000000000000000196ca228159aa"
    ],
   ............
}
```

`constant_result` 是 `balanceOf` 的返回值。以下是解码 `constant_result` 的示例代码：

```javascript
async function main() {
  // 必须以 0x 开头
    let outputs = '0x000000000000000000000000000000000000000000000000000196ca228159aa'
    //
    // ['uint256 '] 是返回值类型列表。如果有多个返回值，按顺序填写类型
    let result = await decodeParams(['uint256'], outputs, false)
    console.log(result)
}

main()
```

示例代码输出：

```
[ BigNumber { _hex: '0x0196ca228159aa' } ]
```

#### 使用 trident-java 进行参数解码示例

**解码数据**

以下 Java 代码使用 trident 解码数据字段，并获取 `transfer` 函数传递的参数：

```java
final String DATA = "a9059cbb0000000000000000000000007fdf5157514bf89ffcb7ff36f34772afd4cdc7440000000000000000000000000000000000000000000000000de0b6b3a7640000";

public void dataDecodingTutorial() {
    String rawSignature = DATA.substring(0, 8);
    String signature = "transfer(address,uint256)"; // 函数签名
    Address rawRecipient = TypeDecoder.decodeAddress(DATA.substring(8, 72)); // 接收地址
    String recipient = rawRecipient.toString();
    Uint256 rawAmount = TypeDecoder.decodeNumeric(DATA.substring(72, 136), Uint256.class); // 数量
    BigInteger amount = rawAmount.getValue();

    System.out.println(signature);
    System.out.println("Transfer " + amount + " to " + recipient);
}
```

**解码合约查询操作的返回值**

常量函数调用将返回一个 `TransactionExtention` 对象，其中 `constantResult` 字段是查询结果，为一个列表。将其转换为十六进制字符串后，可以使用上述示例代码中的 `TypeDecoder` 类解码合约查询操作的返回值。或者也可以使用 `org.tron.trident.abi.FunctionReturnDecoder` 的 `decode` 方法：

在 `org.tron.trident.abi.FunctionReturnDecoder.decode` 方法中指定返回值的类型，可以将结果转换为该类型的对象。

```java
public BigInteger balanceOf(String accountAddr) {
    // 构造函数
    Function balanceOf = new Function("balanceOf",
            Arrays.asList(new Address(accountAddr)), Arrays.asList(new TypeReference<Uint256>() {}));
    // 调用函数
    TransactionExtention txnExt = wrapper.constantCall(Base58Check.bytesToBase58(ownerAddr.toByteArray()), 
            Base58Check.bytesToBase58(cntrAddr.toByteArray()), balanceOf);
    // 将常量结果转换成人类可读文本
    String result = Numeric.toHexString(txnExt.getConstantResult(0).toByteArray());
    return (BigInteger) FunctionReturnDecoder.decode(result, balanceOf.getOutputParameters()).get(0).getValue();
}
```