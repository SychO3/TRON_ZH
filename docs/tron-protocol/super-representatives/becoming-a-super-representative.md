# 成为超级代表
***
任何账户都可以申请成为超级代表候选人，然后参加超级代表选举。
本文将介绍如何成为超级代表并参与区块生产。

## 1. 创建账户
要参加超级代表选举，需要先拥有一个波场网络账户。
如果已经拥有波场账户，请直接跳到第 2 步。

建议离线创建账户，例如，可以使用各种 SDK、wallet-cli 或其他工具。
在本文中，将使用 [wallet-cli](https://github.com/tronprotocol/wallet-cli) 命令行钱包来创建账户地址和发送交易。

**创建账户**

在 wallet-cli 中输入 `RegisterWallet` 命令，然后按提示输入密码。

```shell
wallet> RegisterWallet 
Please input password.
password: 
Please input password again.
password: 
Register a wallet successful, keystore file name is UTC--2024-04-18T07-24-17.307000000Z--TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb.json
wallet> 
```

账户创建成功后，私钥将以 keystore 文件的形式保存在 wallet-cli 运行目录下的 `Wallet` 子文件夹中。
本例中创建的密钥存储文件名为 `UTC--2024-04-18T07-24-17.307000000Z--TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb.json`，
文件名的前半部分是账户创建时间，
后半部分是 Base58 格式的账户地址：`TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb`。

**登录 wallet-cli**

在 wallet-cli 中输入 `login` 命令，然后根据提示输入创建账户时输入的密码。

```shell
wallet> login
Please input your password.
password: 
Login successful !!!
wallet> 
```

登录后，可以通过 wallet-cli 轻松操作该账户和发送交易。

通过 `BackupWallet` 命令查看账户私钥，在此过程中需要输入密码。

!!! tip ""
    切记备份私钥，以保护账户安全。

```shell
wallet> BackupWallet
Please input your password.
password: 
BackupWallet successful !!
8759279fd54e2e141733742029d9d6a1e5fd7c5f7336bdbf46ddd3748f68a5ef
```

## 2. 存入TRX
申请成为超级代表候选人需要支付 `9999TRX` 的申请费，由于所有交易都需要消耗带宽资源，
当账户带宽不足时，需要燃烧 TRX 来支付带宽。
因此，为了确保申请成为超级代表候选人的交易以及以后可能进行的更新账户名称等其他交易的顺利进行，
建议向账户存入 `10100TRX` 。可以通过任何交易所提取 TRX，或将其他钱包中的 TRX 转入该账户。

## 3. 申请成为超级代表候选人
在 wallet-cli 中，通过 `CreateWitness` 命令发送交易，申请成为超级代表候选人。
该命令需要一个表示超级代表网站 URL 的参数。
本例中的网址是 www.your-website.com，操作时请将其替换为实际的网址。

```shell
wallet> CreateWitness www.your-website.com
{
	"raw_data":{
		"contract":[
			{
				"parameter":{
					"value":{
						"owner_address":"TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb",
						"url":"www.your-website.com"
					},
					"type_url":"type.googleapis.com/protocol.WitnessCreateContract"
				},
				"type":"WitnessCreateContract"
			}
		],
		"ref_block_bytes":"d140",
		"ref_block_hash":"fe6d51f5debe3a0d",
		"expiration":1713428409000,
		"timestamp":1713428351865
	},
	"raw_data_hex":"0a02d1402208fe6d51f5debe3a0d40a8d5aa82ef315a67080512630a32747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5769746e657373437265617465436f6e7472616374122d0a15410ffe4eb0f39afcc431fdb22d77d2263161ea210612147777772e796f75722d7765622d7369742e636f6d70f996a782ef31"
}
before sign transaction hex string is 0a85010a02d1402208fe6d51f5debe3a0d40a8d5aa82ef315a67080512630a32747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5769746e657373437265617465436f6e7472616374122d0a15410ffe4eb0f39afcc431fdb22d77d2263161ea210612147777772e796f75722d7765622d7369742e636f6d70f996a782ef31
Please confirm and input your permission id, if input y or Y means default 0, other non-numeric characters will cancel transaction.
```

检查并确认交易无误，然后输入 `y` 并按 `Enter`，然后根据提示输入密码，等待交易完成。
成功后，会看到`CreateWitness success!!`的字样。

```shell
y
Please choose your key for sign.
Please input your password.
password: 
after sign transaction hex string is 0a85010a02d1402208fe6d51f5debe3a0d40cfc7cd8cef315a67080512630a32747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5769746e657373437265617465436f6e7472616374122d0a15410ffe4eb0f39afcc431fdb22d77d2263161ea210612147777772e796f75722d7765622d7369742e636f6d70f996a782ef311241ec56e62858ccf98ff5138002d6d4f1a3eee0c66948704956e2ef8572a8bde78e1ecc72cfa2541ace61076e4a4d4ac1c560cc5d180a28575231414a0a3ea876a701
txid is e58cbd7f8ef280c3e53c36c43e9ae5ad852af6735e439b37f65f9e0744625789
CreateWitness successful !!
wallet> 
```

执行上述交易后，等待一分钟左右再查询该账户是否为超级代表候选账户。
wallet-cli 查询账户信息的命令是 `getaccount`，参数是查询到的超级代表账户地址：

```shell
wallet> getaccount TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb
{
    "address": "TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb",
    ......
    "is_witness": true,
    ......
}
```

如果 `is_witness` 的值为 `true`，则表示该账户是超级代表候选账户。
如果返回结果中没有 `is_witness` 字段，则表示申请成为超级代表候选账户的交易没有成功执行，
也不在链上。

!!!note ""
    超级代表 URL 可以更改。您可以通过 wallet-cli 的 `UpdateWitness` 命令修改 URL。

## 4.（可选）更新经纪比率
经纪比例的默认值为 20，即超级代表成为超级代表候选人后，
默认保留总收入的 20%，并将总收入的 80% 奖励给选民。经纪比例是可以修改的。

假设要修改经纪比例为 100，即全部收入归超级代表所有，在 wallet-cli 中输入 `updateBrokerage` 命令，
同时输入两个参数：超级代表账户地址和经纪比例：

```shell
wallet> updateBrokerage TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb 100
{
	"raw_data":{
		"contract":[
			{
				"parameter":{
					"value":{
						"brokerage":100,
						"owner_address":"410ffe4eb0f39afcc431fdb22d77d2263161ea2106"
					},
					"type_url":"type.googleapis.com/protocol.UpdateBrokerageContract"
				},
				"type":"UpdateBrokerageContract"
			}
		],
		"ref_block_bytes":"d850",
		"ref_block_hash":"63c8641e2cf43e43",
		"expiration":1713434061000,
		"timestamp":1713434002484
	},
	"raw_data_hex":"0a02d850220863c8641e2cf43e4340c8d18385ef315a55083112510a34747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e55706461746542726f6b6572616765436f6e747261637412190a15410ffe4eb0f39afcc431fdb22d77d2263161ea2106106470b4888085ef31"
}
before sign transaction hex string is 0a730a02d850220863c8641e2cf43e4340c8d18385ef315a55083112510a34747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e55706461746542726f6b6572616765436f6e747261637412190a15410ffe4eb0f39afcc431fdb22d77d2263161ea2106106470b4888085ef31
Please confirm and input your permission id, if input y or Y means default 0, other non-numeric characters will cancel transaction.
```

检查并确认交易无误，然后输入 `y` 并按 `Enter`，然后根据提示输入密码，等待交易完成。成功后，会看到`UpdateBrokerage success !!!`的字样。

```shell
y
Please choose your key for sign.
Please input your password.
password: 
after sign transaction hex string is 0a730a02d850220863c8641e2cf43e43409bb8a68fef315a55083112510a34747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e55706461746542726f6b6572616765436f6e747261637412190a15410ffe4eb0f39afcc431fdb22d77d2263161ea2106106470b4888085ef3112417b6e186b3ceb00af97d38267ff5b2379dd981eeb7d972745c81a588e6e37b6111dd005f4434943ad255cbf600e0d37a06a2e48996b490c68fd888974a7d90e3b01
txid is 2020feb596fdbda28b3ba3243978aedf39f8aaafe2e2c8917eb1fd8370a74d1b
UpdateBrokerage successful !!!
```

## 5.（可选）设置超级代表账户名
任何账户都可以设置账户名，但只能设置一次。为了便于社区宣传和推广，建议超级代表设置一个账户名。

!!! info ""
    账户名只能设置一次。

如果要设置账户名，请在 wallet-cli 中输入 `UpdateAccount` 命令，
参数为：账户名。本例中的账户名是 "SR-Name"：

```shell
wallet> UpdateAccount SR-Name
{
	"raw_data":{
		"contract":[
			{
				"parameter":{
					"value":{
						"account_name":"SR-Name",
						"owner_address":"TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb"
					},
					"type_url":"type.googleapis.com/protocol.AccountUpdateContract"
				},
				"type":"AccountUpdateContract"
			}
		],
		"ref_block_bytes":"da37",
		"ref_block_hash":"71fd967d003f6932",
		"expiration":1713435582000,
		"timestamp":1713435522526
	},
	"raw_data_hex":"0a02da37220871fd967d003f693240b0bce085ef315a5a080a12560a32747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e4163636f756e74557064617465436f6e747261637412200a0753522d4e616d651215410ffe4eb0f39afcc431fdb22d77d2263161ea210670deebdc85ef31"
}
before sign transaction hex string is 0a780a02da37220871fd967d003f693240b0bce085ef315a5a080a12560a32747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e4163636f756e74557064617465436f6e747261637412200a0753522d4e616d651215410ffe4eb0f39afcc431fdb22d77d2263161ea210670deebdc85ef31
Please confirm and input your permission id, if input y or Y means default 0, other non-numeric characters will cancel transaction.
```

检查并确认交易无误，然后输入 `y` 并按`Enter`，然后根据提示输入密码，等待交易完成。成功后，将看到`Update Account successful !!!!`字样。

```shell
y
Please choose your key for sign.
Please input your password.
password: 
after sign transaction hex string is 0a780a02da37220871fd967d003f693240bb9b8390ef315a5a080a12560a32747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e4163636f756e74557064617465436f6e747261637412200a0753522d4e616d651215410ffe4eb0f39afcc431fdb22d77d2263161ea210670deebdc85ef3112419fd9833e248e6c814b92179e38de7a0ecaa4cdd221acfdbe4a729c8142d8858a45ceb8dc3cda954188384b68b1a7c603cbfb94ce23982c5812aa563a7a26ee4b00
txid is 57b6c8b53c0a7c61b43030a4cba724e826b9b3b8d1ace60612e13c6861203659
Update Account successful !!!!
```

设置账户名后，可以通过 walllet-cli `getaccount` 命令进行查询。

## 6. 运行超级代表节点
超级代表需要运行波场节点才能参与区块生产，同时也会获得相应的奖励。

节点构建过程如下：

1. 下载最新数据快照
2. 部署节点 - 节点部署指南

启动节点后，如何检查节点同步是否完成？
您可以通过 `/wallet/getnowblock `或 `wallet/getnodeinfo `接口查询本地节点的最新区块高度，
并与 [TRONSCAN](https://tronscan.org/#/blockchain/blocks) 上显示的结果进行比较。
如果两者一致，则表示本地节点已完成区块同步，状态正常，
可以进行交易验证和广播、区块处理和生产。
请注意，只有当超级代表候选人账户的得票数排在前 27 位时，即候选人成为超级代表后，节点才会生产区块。

## 7.（可选）参与提案的创建和投票
波场网络的升级和管理需要通过提案来完成。
一个提案可以修改一个或多个链上参数。
每个超级代表、超级代表合作伙伴和超级代表候选人都有权发起修改波场网络参数的提案，
但只有超级代表有权对提案进行投票。

### 发起提案
在 wallet-cli 中输入 `CreateProposal` 命令，并输入动态参数编号及其值。在本例中，参数 70 的值修改为 15。

```shell
wallet> CreateProposal 70 15
{
	"raw_data":{
		"contract":[
			{
				"parameter":{
					"value":{
						"owner_address":"TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb",
						"parameters":[
							{
								"value":15,
								"key":70
							}
						]
					},
					"type_url":"type.googleapis.com/protocol.ProposalCreateContract"
				},
				"type":"ProposalCreateContract"
			}
		],
		"ref_block_bytes":"2d3e",
		"ref_block_hash":"a672a46dfaa0c5dc",
		"expiration":1713502020000,
		"timestamp":1713501961789
	},
	"raw_data_hex":"0a022d3e2208a672a46dfaa0c5dc40a0c3b7a5ef315a58081012540a33747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e50726f706f73616c437265617465436f6e7472616374121d0a15410ffe4eb0f39afcc431fdb22d77d2263161ea210612040846100f70bdfcb3a5ef31"
}
before sign transaction hex string is 0a760a022d3e2208a672a46dfaa0c5dc40a0c3b7a5ef315a58081012540a33747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e50726f706f73616c437265617465436f6e7472616374121d0a15410ffe4eb0f39afcc431fdb22d77d2263161ea210612040846100f70bdfcb3a5ef31
Please confirm and input your permission id, if input y or Y means default 0, other non-numeric characters will cancel transaction.
```

检查并确认交易无误后，输入 `y` 并按 `Enter`，然后根据提示输入密码，等待交易完成。
成功后，您将看到 `CreateProposal successful !!!` 字样。

```shell
y
Please choose your key for sign.
Please input your password.
password: 
after sign transaction hex string is 0a760a022d3e2208a672a46dfaa0c5dc40b1acdaafef315a58081012540a33747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e50726f706f73616c437265617465436f6e7472616374121d0a15410ffe4eb0f39afcc431fdb22d77d2263161ea210612040846100f70bdfcb3a5ef311241b13d61162cb44bd1a8cc7e2129a93cb7650a39bd679a48d3a7052abef9d9d29736158be610214e242c8059e205bfe50c572c4d8fc1ae37177ddf373f09cf87bf01
txid is 854da5d9d2577894a061a2b3aee46075532d40ac2f047a699887c00bf2d6cbf1
CreateProposal successful !!
```

### 超级代表提案投票
在 wallet-cli 中输入 `ApproveProposal` 命令，并输入提案 ID 和 true。
可以使用 `ListProposals`（列表提案）命令查看刚刚创建的提案的 ID。
在本例中，刚刚创建的提案 ID 是 19602。

```shell
wallet> ApproveProposal 19602 true
{
	"raw_data":{
		"contract":[
			{
				"parameter":{
					"value":{
						"proposal_id":19602,
						"is_add_approval":true,
						"owner_address":"TBRmnXKMEVfQ8XeQA2NroC9cGi77TvPbNb"
					},
					"type_url":"type.googleapis.com/protocol.ProposalApproveContract"
				},
				"type":"ProposalApproveContract"
			}
		],
		"ref_block_bytes":"2e24",
		"ref_block_hash":"158e49f435409805",
		"expiration":1713502734000,
		"timestamp":1713502675384
	},
	"raw_data_hex":"0a022e242208158e49f43540980540b08de3a5ef315a59081112550a34747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e50726f706f73616c417070726f7665436f6e7472616374121d0a15410ffe4eb0f39afcc431fdb22d77d2263161ea210610929901180170b8c3dfa5ef31"
}
before sign transaction hex string is 0a770a022e242208158e49f43540980540b08de3a5ef315a59081112550a34747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e50726f706f73616c417070726f7665436f6e7472616374121d0a15410ffe4eb0f39afcc431fdb22d77d2263161ea210610929901180170b8c3dfa5ef31
Please confirm and input your permission id, if input y or Y means default 0, other non-numeric characters will cancel transaction.
```

检查并确认交易无误，然后输入 `y 并按` `Enter`，然后根据提示输入密码，等待交易完成。成功后，将看到 `ApproveProposal successful !!!` 字样。

```shell
y
Please choose your key for sign.
Please input your password.
password: 
after sign transaction hex string is 0a770a022e242208158e49f435409805408cf385b0ef315a59081112550a34747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e50726f706f73616c417070726f7665436f6e7472616374121d0a15410ffe4eb0f39afcc431fdb22d77d2263161ea210610929901180170b8c3dfa5ef3112416653d14145ea0a861cdbb41d38fb5e46023cfa5dde9ce523e87c75ceb86e4dbe6e93b4fe2bb2c32f40812dbea40a551fe490fefd7aeb73d4ec55cbe7a71c924c00
txid is 3c3262aaee16e58bacc9f345a0422e08972a446c5859807e11d9b32f19f02afc
ApproveProposal successful !!!
```