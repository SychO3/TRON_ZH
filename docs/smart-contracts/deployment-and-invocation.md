# 部署与调用

***

## 智能合约的部署

部署智能合约是创建一个 `CreateSmartContract` 类型的交易，可以通过全节点 API 的 `wallet/deploycontract` 来创建，创建完成后再对交易进行签名并广播：

```shell
curl --request POST \
     --url https://api.shasta.trongrid.io/wallet/deploycontract \
     --header 'Accept: application/json' \
     --header 'Content-Type: application/json' \
     --data '
{
     "owner_address": "41D1E7A6BC354106CB410E65FF8B181C600FF14292",
     "abi": "[{\"constant\":false,\"inputs\":[{\"name\":\"key\",\"type\":\"uint256\"},{\"name\":\"value\",\"type\":\"uint256\"}],\"name\":\"set\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"constant\":true,\"inputs\":[{\"name\":\"key\",\"type\":\"uint256\"}],\"name\":\"get\",\"outputs\":[{\"name\":\"value\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"}]",
     "bytecode": "608060405234801561001057600080fd5b5060de8061001f6000396000f30060806040526004361060485763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416631ab06ee58114604d5780639507d39a146067575b600080fd5b348015605857600080fd5b506065600435602435608e565b005b348015607257600080fd5b50607c60043560a0565b60408051918252519081900360200190f35b60009182526020829052604090912055565b600090815260208190526040902054905600a165627a7a72305820fdfe832221d60dd582b4526afa20518b98c2e1cb0054653053a844cf265b25040029",
     "fee_limit": 1000000,
     "origin_energy_limit": 100000,
     "name": "SomeContract",
     "call_value": 0,
     "consume_user_resource_percent": 100
}
'
```

返回结果：

```json
{
  "visible": false,
  "txID": "e4eb4df3a64a33565059ee3cef29b93b79f58ece9e7e8153fda3bbfe40fb0524",
  "contract_address": "416c5d359d1836085cdad65788e1ce53d3d7a13dd6",
  "raw_data": {
    "contract": [
      {
        "parameter": {
          "value": {
            "owner_address": "41d1e7a6bc354106cb410e65ff8b181c600ff14292",
            "new_contract": {
              "bytecode": "608060405234801561001057600080fd5b5060de8061001f6000396000f30060806040526004361060485763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416631ab06ee58114604d5780639507d39a146067575b600080fd5b348015605857600080fd5b506065600435602435608e565b005b348015607257600080fd5b50607c60043560a0565b60408051918252519081900360200190f35b60009182526020829052604090912055565b600090815260208190526040902054905600a165627a7a72305820fdfe832221d60dd582b4526afa20518b98c2e1cb0054653053a844cf265b25040029",
              "consume_user_resource_percent": 100,
              "name": "SomeContract",
              "origin_address": "41d1e7a6bc354106cb410e65ff8b181c600ff14292",
              "abi": {
                "entrys": [
                  {
                    "inputs": [
                      {
                        "name": "key",
                        "type": "uint256"
                      },
                      {
                        "name": "value",
                        "type": "uint256"
                      }
                    ],
                    "name": "set",
                    "stateMutability": "Nonpayable",
                    "type": "Function"
                  },
                  {
                    "outputs": [
                      {
                        "name": "value",
                        "type": "uint256"
                      }
                    ],
                    "constant": true,
                    "inputs": [
                      {
                        "name": "key",
                        "type": "uint256"
                      }
                    ],
                    "name": "get",
                    "stateMutability": "View",
                    "type": "Function"
                  }
                ]
              },
              "origin_energy_limit": 100000
            }
          },
          "type_url": "type.googleapis.com/protocol.CreateSmartContract"
        },
        "type": "CreateSmartContract"
      }
    ],
    "ref_block_bytes": "0a49",
    "ref_block_hash": "975853d6629c8702",
    "expiration": 1652153760000,
    "fee_limit": 1000000,
    "timestamp": 1652153701556
  },
  "raw_data_hex": "0a020a492208975853d6629c87024080f2a6e08a305add03081e12d8030a30747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e437265617465536d617274436f6e747261637412a3030a1541d1e7a6bc354106cb410e65ff8b181c600ff142921289030a1541d1e7a6bc354106cb410e65ff8b181c600ff142921a5c0a2b1a03736574220e12036b65791a0775696e743235362210120576616c75651a0775696e74323536300240030a2d10011a03676574220e12036b65791a0775696e743235362a10120576616c75651a0775696e743235363002400222fd01608060405234801561001057600080fd5b5060de8061001f6000396000f30060806040526004361060485763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416631ab06ee58114604d5780639507d39a146067575b600080fd5b348015605857600080fd5b506065600435602435608e565b005b348015607257600080fd5b50607c60043560a0565b60408051918252519081900360200190f35b60009182526020829052604090912055565b600090815260208190526040902054905600a165627a7a72305820fdfe832221d60dd582b4526afa20518b98c2e1cb0054653053a844cf265b2504002930643a0c536f6d65436f6e747261637440a08d0670b4a9a3e08a309001c0843d"
}
```

不同类型交易的[交易结构](../tron-protocol/transaction.md)相同，但 `raw_data` 中包含的内容不同，主要体现在交易的 `contract.parameter.value` 字段。

合约部署交易中的 `contract.parameter.value` 包含以下内容：

- `owner_address`：合约所有者的地址，即合约部署者地址
- `new_contract`：新的智能合约的详细信息
    - `origin_address`：智能合约所有者的地址
    - `contract_address`：智能合约的地址
    - `ABI`：智能合约的 ABI
    - `bytecode`：智能合约的字节码
    - `call_value`：发送给智能合约的 TRX 数量
    - `consume_user_resource_percent`：用户能量支付的百分比
    - `name`：智能合约的名称
    - `origin_energy_limit`：每笔交易所有者能量消耗的限制，以 `sun` 为单位
- `call_token_value`：发送给新创建智能合约的 TRC10 通证数量
- `token_id`：TRC10 通证 ID

## 智能合约调用与查询

### triggersmartcontract

合约调用是创建一个 `TriggerSmartContract` 类型的交易。你可以通过全节点 API `wallet/triggersmartcontract` 创建一个触发合约调用交易，交易创建后，需要签名并广播到整个网络。

```shell
curl -X POST https://api.shasta.trongrid.io/wallet/triggersmartcontract -d '{
"contract_address":"419E62BE7F4F103C36507CB2A753418791B1CDC182",
"function_selector":"transfer(address,uint256)",
"parameter":"00000000000000000000004115208EF33A926919ED270E2FA61367B2DA3753DA0000000000000000000000000000000000000000000000000000000000000032",
"fee_limit":100000000,
"call_value":0,
"owner_address":"41977C20977F412C2A1AA4EF3D49FEE5EC4C31CDFB"
}'
```

参数说明：

- `contract_address`：合约地址
- `owner_address`：调用者地址
- `function_selector`：合约函数
- `parameter`：合约方法的编码参数值。在此示例中，需要传入两个参数，即 `address` 和 `uint256` 类型。关于如何编码和解码参数的详细信息，请参考“参数编码与解码”章节
- `fee_limit`：调用者愿意承担的交易能量消耗上限，请参考 `fee_limit` 的描述
- `call_value`：发送给智能合约的 TRX 数量

成功执行上述命令后，将返回如下结果，其中包括一个合约调用交易：`transaction`:

```json
{
	"result": {
		"result": true
	},
	"transaction": {
		"visible": false,
		"txID": "d2ce86097df40287ad45ebc67f0d546ee98c2d7cd7c101e4d4d5b0c8a752d900",
		"raw_data": {
			"contract": [{
				"parameter": {
					"value": {
						"data": "a9059cbb00000000000000000000004115208ef33a926919ed270e2fa61367b2da3753da0000000000000000000000000000000000000000000000000000000000000032",
						"owner_address": "41977c20977f412c2a1aa4ef3d49fee5ec4c31cdfb",
						"contract_address": "419e62be7f4f103c36507cb2a753418791b1cdc182"
					},
					"type_url": "type.googleapis.com/protocol.TriggerSmartContract"
				},
				"type": "TriggerSmartContract"
			}],
			"ref_block_bytes": "1c51",
			"ref_block_hash": "74912b480b7b887c",
			"expiration": 1652169501000,
			"fee_limit": 100000000,
			"timestamp": 1652169442098
		},
		"raw_data_hex": "0a021c51220874912b480b7b887c40c8d2e7e78a305aae01081f12a9010a31747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e54726967676572536d617274436f6e747261637412740a1541977c20977f412c2a1aa4ef3d49fee5ec4c31cdfb1215419e62be7f4f103c36507cb2a753418791b1cdc1822244a9059cbb00000000000000000000004115208ef33a926919ed270e2fa61367b2da3753da000000000000000000000000000000000000000000000000000000000000003270b286e4e78a30900180c2d72f"
	}
}
```

合约调用交易中的 `contract.parameter.value `包含以下内容：

- `owner_address`：调用者地址
- `contract_address`：合约地址
- `data`：合约函数选择器及其参数。前4字节是合约函数选择器, 是对合约函数名称和参数进行 `Keccak-256` 操作所得结果的前4字节，用于虚拟机查找函数。数据的剩余字节是函数的参数。关于编码和解码的详细信息，请参考“参数编码与解码”章节
- `call_value`：发送给智能合约的 TRX 数量
- `token_id`：发送给合约的 TRC10 通证的 ID
- `call_token_value`：发送给智能合约的 TRC10 通证数量

### triggerconstantcontract

通过全节点 API `wallet/triggerconstantcontract` 调用合约的常量函数。由于这是一个查询操作，不需要上链，因此无需签名和广播：

```shell
curl -X POST https://127.0.0.1:8090/wallet/triggerconstantcontract -d '{
"contract_address":"419E62BE7F4F103C36507CB2A753418791B1CDC182",
"function_selector":"balanceOf(address)",
"parameter":"000000000000000000000041977C20977F412C2A1AA4EF3D49FEE5EC4C31CDFB",
"owner_address":"41977C20977F412C2A1AA4EF3D49FEE5EC4C31CDFB"
}'
```

参数说明：

- `contract_address`：合约地址
- `owner_address`：调用者地址
- `function_selector`：触发的合约方法
- `parameter`：需要传入合约方法的参数。关于编码和解码的详细信息，请参考“参数编码与解码”章节

返回结果如下：

```json
{
	"result": {
		"result": true
	},
	"constant_result": ["0000000000000000000000000000000000000000000000000000000430e1b700"],
	"transaction": {
		"ret": [{}],
		"visible": false,
		"txID": "7f47212aed2fdab232195feece54fc302bde2ce379e92ffd0d0e95206ce7a3bb",
		"raw_data": {
			"contract": [{
				"parameter": {
					"value": {
						"data": "70a08231000000000000000000000041977c20977f412c2a1aa4ef3d49fee5ec4c31cdfb",
						"owner_address": "41977c20977f412c2a1aa4ef3d49fee5ec4c31cdfb",
						"contract_address": "419e62be7f4f103c36507cb2a753418791b1cdc182"
					},
					"type_url": "type.googleapis.com/protocol.TriggerSmartContract"
				},
				"type": "TriggerSmartContract"
			}],
			"ref_block_bytes": "5ab1",
			"ref_block_hash": "f24f075df912f43e",
			"expiration": 1590382815000,
			"timestamp": 1590382762536
		},
		"raw_data_hex": "0a025ab12208f24f075df912f43e4098cecfd1a42e5a8e01081f1289010a31747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e54726967676572536d617274436f6e747261637412540a1541977c20977f412c2a1aa4ef3d49fee5ec4c31cdfb1215419e62be7f4f103c36507cb2a753418791b1cdc182222470a08231000000000000000000000041977c20977f412c2a1aa4ef3d49fee5ec4c31cdfb70a8b4ccd1a42e"
	}
}
```

- constant_result：这是查询合约的结果。在此示例中，返回值类型为 uint256。关于如何对返回值进行编码和解码的详细信息，请参考“参数编码与解码”章节。


## 消耗用户资源百分比

合约调用需要支付一定的资源费用。为了鼓励用户进行合约交易并降低其调用成本，波场网络支持合约部署者分担部分合约调用费用。在部署合约时，可以通过参数 `consume_user_resource_percent` 设置用户的能量支付比例。

用户能量支付比例是用户为智能合约执行支付的能量占比。例如，如果用户能量支付比例设置为 60，用户支付合约执行所需能量的 60%，开发者（合约部署者）支付剩余 40% 的能量。此参数只能是 0 到 100 之间的整数（包括 0 和 100）。建议开发者适当提高用户的能量支付比例，以防止用户攻击合约并耗尽合约拥有者的账户资源。

合约成功部署后，用户能量支付比例也可以修改。例如，合约拥有者可以通过全节点 HTTP API `wallet/updatesetting` 进行修改。

注意：虽然合约开发者可能需要分担一定比例的合约调用能量费用，但当开发者账户的能量不足以支付其需要承担的部分，或此调用消耗的开发者账户能量超过 `origin_energy_limit` 的值时，剩余部分由调用者承担。

## 费用限制

`feelimit` 指的是调用者愿意为智能合约的部署或调用承担的能量费用上限，以 `sun` 为单位（`1TRX = 1e6 sun`）。目前，能量费用上限可以设置为 `15000TRX`。如果 `feelimit` 设置大于 15000 TRX，将会产生错误。

在执行合约时，能量按照每条指令逐一计算和扣除。如果使用的能量超出上限，合约执行将失败且扣除的能量不会被返还。

在将合约部署到主网之前，最好设置合理的 `feelimit`。例如，在部署大型合约或运行复杂功能时，需要更大的 `feelimit`。然而，由于合约执行超时、合约中的无限循环、非法操作和向不存在的账户转账等情况的存在，较低的 `feelimit` 会是更好的选择。