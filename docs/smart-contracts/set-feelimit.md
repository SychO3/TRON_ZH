# FeeLimit 参数设置
***
`FeeLimit` 是智能合约交易的一个参数,用于设置调用者在合约部署或调用时愿意承担的能量成本上限,单位是 sun (`1TRX = 1e6 sun`)。默认值为 `0`。目前,最大可设的 `FeeLimit` 上限是 `15000TRX`。

在执行合约时,能量会按指令逐一计算和扣除。如果超出能量使用量,合约执行将失败,已扣除的能量不会退还。因此,在部署或调用合约之前,建议设置一个合适的 FeeLimit,以确保智能合约交易的正常执行。以下描述如何估算智能合约交易的能量消耗以及设置 `FeeLimit` 参数。

## 如何确定 FeeLimit 参数？

由于动态能量模型机制的存在,热门合约的能量消耗是动态变化的。
因此,在不同的时间段调用同一个合约函数可能会导致不同的能量消耗。
在不同的时间段,调用热门合约的交易需要设置不同的 `FeeLimit` 参数。这里介绍三种设置 FeeLimit 的方法：

1. 在每次合约调用前估算能量

    在每次交易发送前,通过 API 估算交易的总能量消耗,并根据估算的能量消耗确定交易的 `FeeLimit` 参数：

    ```
    智能合约交易的 FeeLimit = 估算的总能量消耗 * EnergyPrice
    ```

2. 每个维护周期获取一次合约的 `energy_factor`

    首先,通过 `triggerconstantcontract` API 或合约的历史经验确定合约某个函数的基础能量消耗,然后在每个维护周期获取合约的 energy_factor 参数：

    ```
    智能合约交易的 FeeLimit = 估算的基础能量消耗 * (1 + energy_factor) * EnergyPrice
    ```

3. 根据 `max_factor` 设置 FeeLimit

    首先,通过 `triggerconstantcontract` API 或合约的历史经验确定合约某个函数的基础能量消耗,然后获取链上的 `max_factor` 参数。`max_factor` 是能量惩罚系数的最大比率,因此无论热门合约的能量消耗如何波动,都不会超过这个最大比率：

    ```
    智能合约交易的 FeeLimit = 估算的基础能量消耗 * (1 + max_factor) * EnergyPrice
    ```

上述第一种方法的优点是 `FeeLimit` 设置非常精准，但缺点是操作较为繁琐,每笔交易都需要进行估算。与第一种方法相比,第二种方法保持了 `FeeLimit` 设置的准确性,但仍然需要每个维护周期(6小时)获取合约的 `energy_factor` 参数。第三种方法的优点是操作简便,不需要频繁获取 `max_factor` 参数,但计算出的 `FeeLimit` 会大于实际能量成本,因为大多数合约的 `energy_factor` 不会达到 `max_factor`。


## 如何估算能量消耗？

开发者可以调用 `wallet/triggerconstantcontract` API 来估算合约调用或部署交易的能量消耗值。我们通过一个例子来说明：

如何估算合约调用交易的能量消耗

```bash
$ curl -X POST https://nile.trongrid.io/wallet/triggerconstantcontract -d '{
    "owner_address": "TTGhREx2pDSxFX555NWz1YwGpiBVPvQA7e",
    "contract_address": "TVSvjZdyDSNocHm7dP3jvCmMNsCnMTPa5W",
    "function_selector": "transfer(address,uint256)",
    "parameter": "0000000000000000000000002ce5de57373427f799cc0a3dd03b841322514a8c00000000000000000000000000000000000000000000000000038d7ea4c68000",
    "visible": true
}'
```

返回结果为：

```json
{
   ……
   "result": {
       "result": true
   },
   "energy_used": 46236,
   "energy_penalty": 32983,
   ……
}
```

该示例中 `result.result=true` 表示估算操作成功执行，`energy_used` 的值是交易的估算能量消耗，其中基本能量消耗值为 `(energy_used - energy_penalty)`，`energy_penalty` 的值是额外能量消耗。如果 `result.result` 字段为 `false`，则表示估算失败。在再次进行估算之前，请检查交易数据或者合约和账户信息是否正确。


## 如何估算合约部署交易的能量消耗

对于构造函数没有参数的合约，在估算其部署所需的能量时，只需将合约的字节码放入 `wallet/triggerconstantcontract` 接口的数据字段中即可。然而，对于构造函数有参数的合约，在部署时还需传入参数。这些参数同样通过数据字段传入：参数经过 ABI 编码后放在合约字节码的后面。请参考以下示例。

估算构造函数没有参数的合约部署的能量消耗。合约示例如下：

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

contract SimpleContract {
    uint storedData;
    
    function set(uint x) public {
        storedData = x;
    }

    function get() public view returns (uint) {
        return storedData;
    }
}
```

编译合约后，我们获得合约字节码：

```
608060405234801561000f575f80fd5b50d3801561001b575f80fd5b50d28015610027575f80fd5b5061015b806100355f395ff3fe608060405234801561000f575f80fd5b50d3801561001b575f80fd5b50d28015610027575f80fd5b506004361061004c575f3560e01c806360fe47b1146100505780636d4ce63c1461006c575b5f80fd5b61006a600480360381019061006591906100d2565b61008a565b005b610074610093565b604051610081919061010c565b60405180910390f35b805f8190555050565b5f8054905090565b5f80fd5b5f819050919050565b6100b18161009f565b81146100bb575f80fd5b50565b5f813590506100cc816100a8565b92915050565b5f602082840312156100e7576100e661009b565b5b5f6100f4848285016100be565b91505092915050565b6101068161009f565b82525050565b5f60208201905061011f5f8301846100fd565b9291505056fea26474726f6e58221220ca11b5749b47f126a08ed4dd6de453cf3e3e1d68c1105af0acdd8a38c18b37ac64736f6c63430008140033
```

将合约字节码放入 `wallet/triggerconstantcontract` 接口的数据字段中。估算命令为：

```bash
curl --request POST \
 --url https://api.shasta.trongrid.io/wallet/triggerconstantcontract \
 --header 'accept: application/json' \
 --header 'content-type: application/json' \
 --data '
{
  "owner_address": "TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g",
  "data":"608060405234801561000f575f80fd5b50d3801561001b575f80fd5b50d28015610027575f80fd5b5061015b806100355f395ff3fe608060405234801561000f575f80fd5b50d3801561001b575f80fd5b50d28015610027575f80fd5b506004361061004c575f3560e01c806360fe47b1146100505780636d4ce63c1461006c575b5f80fd5b61006a600480360381019061006591906100d2565b61008a565b005b610074610093565b604051610081919061010c565b60405180910390f35b805f8190555050565b5f8054905090565b5f80fd5b5f819050919050565b6100b18161009f565b81146100bb575f80fd5b50565b5f813590506100cc816100a8565b92915050565b5f602082840312156100e7576100e661009b565b5b5f6100f4848285016100be565b91505092915050565b6101068161009f565b82525050565b5f60208201905061011f5f8301846100fd565b9291505056fea26474726f6e58221220ca11b5749b47f126a08ed4dd6de453cf3e3e1d68c1105af0acdd8a38c18b37ac64736f6c63430008140033",
  "visible": true
}'
```

返回结果为：

```json
{
	"result": {
		"result": true
	},
	"energy_used": 69558,
	......
}
```

估算构造函数有参数的合约部署的能量消耗。合约示例如下：

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.20;

contract SimpleContract {
    uint storedData;
    uint public num;

    constructor(uint _num) {
        num = _num;
    }
    
    function set(uint x) public {
        storedData = x;
    }

    function get() public view returns (uint) {
        return storedData;
    }
}
```

编译合约后，我们获得合约字节码：

```
608060405234801561000f575f80fd5b50d3801561001b575f80fd5b50d28015610027575f80fd5b5060405161024f38038061024f8339818101604052810190610049919061008d565b80600181905550506100b8565b5f80fd5b5f819050919050565b61006c8161005a565b8114610076575f80fd5b50565b5f8151905061008781610063565b92915050565b5f602082840312156100a2576100a1610056565b5b5f6100af84828501610079565b91505092915050565b61018a806100c55f395ff3fe608060405234801561000f575f80fd5b50d3801561001b575f80fd5b50d28015610027575f80fd5b5060043610610057575f3560e01c80634e70b1dc1461005b57806360fe47b1146100795780636d4ce63c14610095575b5f80fd5b6100636100b3565b60405161007091906100e2565b60405180910390f35b610093600480360381019061008e9190610129565b6100b9565b005b61009d6100c2565b6040516100aa91906100e2565b60405180910390f35b60015481565b805f8190555050565b5f8054905090565b5f819050919050565b6100dc816100ca565b82525050565b5f6020820190506100f55f8301846100d3565b92915050565b5f80fd5b610108816100ca565b8114610112575f80fd5b50565b5f81359050610123816100ff565b92915050565b5f6020828403121561013e5761013d6100fb565b5b5f61014b84828501610115565b9150509291505056fea26474726f6e582212205d0adb1a1985b9d6f432a9defc416a17dfe1c931bfb8554978c0f37cfe4cc99f64736f6c63430008140033
```

此合约在部署时需要传入一个 `uint` 类型的参数。假设在部署时传入的值为 `2`，经过 ABI 编码后为 `000000000000000000000000000000000000000000000000000000000000000000000002`，将其放在字节码的后面以获取数据字段的值。估算命令为：

```bash
curl --request POST \
 --url https://api.shasta.trongrid.io/wallet/triggerconstantcontract \
 --header 'accept: application/json' \
 --header 'content-type: application/json' \
 --data '
{
  "owner_address": "TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g",
  "data":"608060405234801561000f575f80fd5b50d3801561001b575f80fd5b50d28015610027575f80fd5b5060405161024f38038061024f8339818101604052810190610049919061008d565b80600181905550506100b8565b5f80fd5b5f819050919050565b61006c8161005a565b8114610076575f80fd5b50565b5f8151905061008781610063565b92915050565b5f602082840312156100a2576100a1610056565b5b5f6100af84828501610079565b91505092915050565b61018a806100c55f395ff3fe608060405234801561000f575f80fd5b50d3801561001b575f80fd5b50d28015610027575f80fd5b5060043610610057575f3560e01c80634e70b1dc1461005b57806360fe47b1146100795780636d4ce63c14610095575b5f80fd5b6100636100b3565b60405161007091906100e2565b60405180910390f35b610093600480360381019061008e9190610129565b6100b9565b005b61009d6100c2565b6040516100aa91906100e2565b60405180910390f35b60015481565b805f8190555050565b5f8054905090565b5f819050919050565b6100dc816100ca565b82525050565b5f6020820190506100f55f8301846100d3565b92915050565b5f80fd5b610108816100ca565b8114610112575f80fd5b50565b5f81359050610123816100ff565b92915050565b5f6020828403121561013e5761013d6100fb565b5b5f61014b84828501610115565b9150509291505056fea26474726f6e582212205d0adb1a1985b9d6f432a9defc416a17dfe1c931bfb8554978c0f37cfe4cc99f64736f6c634300081400330000000000000000000000000000000000000000000000000000000000000002",
  "visible": true
}'
```

返回结果为：

```json
{
	"result": {
		"result": true
	},
	"energy_used": 99278,
	......
}
```

在上述返回值中，如果 `result.result` 字段为 `true`，表示估算操作成功执行，`energy_used` 是交易能量消耗的估算值。如果 `result.result` 字段为 `false`，表示估算失败。在进行估算之前，请检查交易数据是否正确。

!!! note
    `triggerconstantcontract` API 可以用于估算链上大多数智能合约调用的能量消耗值，如 USDD、USDT、USDC、TUSD 等。同时，在 Java-tron 4.7.0.1 版本中，新增了 `wallet/estimateenergy` API。与现有的 `wallet/triggerconstantcontract` API 相比，新的 API 在估算少数特殊合约调用时更为准确。但对于全节点来说，启用 `wallet/estimateEnergy` API 是可选的。因此请注意，当开发者调用 `wallet/estimateEnergy` 时，如果错误信息显示节点不支持此功能（此节点不支持估算能量），建议继续使用 `wallet/triggerconstantcontract` API 来估算能量消耗。

以下是一个示例：

```bash
$ curl -X POST https://nile.trongrid.io/wallet/estimateenergy -d '{
    "owner_address": "TTGhREx2pDSxFX555NWz1YwGpiBVPvQA7e",
    "contract_address": "TVSvjZdyDSNocHm7dP3jvCmMNsCnMTPa5W",
    "function_selector": "transfer(address,uint256)",
    "parameter": "0000000000000000000000002ce5de57373427f799cc0a3dd03b841322514a8c00000000000000000000000000000000000000000000000000038d7ea4c68000",
    "visible": true
}'
```

返回结果为：

```json
{
   "result": {
      "result": true
   },
   "energy_required": 34830
}
```

该示例中的 `result.result = true` 表示估算操作成功执行，`energy_required` 的值是交易的估算能量消耗，它包含基本能量消耗与额外能量消耗。