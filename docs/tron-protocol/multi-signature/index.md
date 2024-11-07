# 多重签名
***
## 介绍
多重签名功能允许权限分级，每个权限可对应多个私钥。这使得多人共同控制账户成为可能。
本指南将引导用户了解 TRON 的多重签名实施和设计，请参阅[TIP-16](https://github.com/tronprotocol/TIPs/blob/master/tip-16.md)以获取更多信息。

## 设计
该方案包括所有者、期限和活动权限三个权限级别。所有者权限拥有执行所有合同的权力，期限权限用于超级代表，而活动权限是一种自定义权限（可与权限集结合使用）。

### 结构描述
#### 1.账户修改
```javascript
message Account { 
   ... 
   Permission owner_permission = 31;
   Permission witness_permission = 32;
   repeated Permission active_permission = 33;
 }
```
账户结构中添加了三个权限属性，即 owner_permission、witness_permission 和 active_permission，其中 active_permission 是一个列表，最多可指定 8 个。

#### 2.合约类型修改
```javascript
message Transaction {
   message Contract {
     enum ContractType { 
       AccountCreateContract = 0; 
       ... 
       AccountPermissionUpdateContract = 46; 
       }
     }  
   }
 }
```
新增了一个事务类型 AccountPermissionUpdateContract，用于更新账户权限。

#### 3.账户权限更新合同
```javascript
message AccountPermissionUpdateContract {
   bytes owner_address = 1;
   Permission owner = 2;   
   Permission witness = 3; 
   repeated Permission actives = 4; 
 }
```


| 参数            | 描述                         |
|:--------------|:---------------------------|
| owner_address | 要修改的账户地址                   |
| owner         | 修改所有者权限                    |
| witness       | 修改后的超级代表许可（如果是超级代表）        |
| actives       | 修改后的活动许可 |

该界面会覆盖原始账户权限，因此如果只想修改所有者权限，还需要设置超级代表（如果是超级代表账户）和激活者。

#### 4.权限
```javascript
message Permission {
   enum PermissionType {
     Owner = 0;
     Witness = 1;
     Active = 2;
   }
   PermissionType type = 1; 
   int32 id = 2;     
   string permission_name = 3;
   int64 threshold = 4;
   int32 parent_id = 5; 
   bytes operations = 6;  
   repeated Key keys = 7;// 
 }
```


| 参数              | 描述                                                                                     |
|:----------------|:---------------------------------------------------------------------------------------|
| PermissionType  | 权限类型，目前只支持三种权限。                                                                        |
| id              | 该值由系统自动设置，所有者 id=0 和见证人 id=1。活动 id 从 2 开始递增。执行合同时，id 用于指定使用哪个权限。例如，如果使用所有者权限，id 将设为 0。 |
| permission_name | 权限名称，由用户设置，长度限制为 32 字节。                                                                |
| threshold       | 阈值，只有当参与签名的权重之和超过域值时，才允许进行相应操作。要求最大值小于长类型。                                             |
| parent_id       | 目前只有 0                                                                                 |
| operations      | 共有 32 个字节（256 位），每个字节代表一个合同的权限，1 表示拥有该合同的权限： "活动权限中的操作示例"                              |
| keys            | 共同拥有权限的地址和权重最多可达 5 个键。                                                                 |


#### 5.key
```javascript
message Key {
   bytes address = 1;
   int64 weight = 2;
 }
```
| 参数      | 描述                                         |
|:--------|:-------------------------------------------|
| address | 具有此特权的地址                                   |
| weight  | 该地址对该许可具有权重 |

#### 6.交易修改
```javascript
message Transaction {
      ...
     int32 Permission_id = 5;
 }
```
在事务中添加与 Permission.id 相对应的 Permission_id 字段，指定使用哪个权限。默认值为 0，即所有者权限。不允许为 1，因为见证权限只用于创建区块，不用于签署事务。

### 拥有者权限
OwnerPermission 是账户的最高权限，用于控制用户的所有权、调整权限结构，Owner 权限还可以执行所有合同。

拥有者权限具有以下特点：

1. OwnerPermission 地址可由 OwnerPermission 修改。
2. 当 OwnerPermission 为空时，账户地址默认为拥有所有者权限。
3. 新创建账户时，账户地址会自动填写在 OwnerPermission 中，默认域值为 1，密钥中包含唯一地址，权重为 1。
4. 如果在执行合同时没有指定 permissionId，则默认使用 OwnerPermission。

### 超级代表权限
超级代表可以使用此权限管理区块节点。非超级代表账户没有此权限。

使用场景示例： 
超级代表在云服务器上部署阻止程序。
为了账户安全，可以将拦截权限分配给另一个地址。
由于该地址只有出站权限，没有 TRX 导出权限，即使服务器上的私钥泄露，TRX 也不会丢失。

超级代表生产节点配置：

1. 在不修改超级代表权限的情况下，无需进行特殊配置。
2. 修改为超级代表权限的区块节点需要重新配置。配置项目如下：

```javascript
#config.conf

// Optional.The default is empty.
// It is used when the witness account has set the witnessPermission.
// When it is not empty, the localWitnessAccountAddress represents the address of the witness account,
// and the localwitness is configured with the private key of the witnessPermissionAddress in the witness account.
// When it is empty,the localwitness is configured with the private key of the witness account.
// Optional, default is empty.
// Used to set the durationPermission when the witness account is set.
// When the value is not empty, localWitnessAccountAddress represents the address of the witness account, and localwitness is the private key of the address in the durationPermission.
// When the value is empty, localwitness is configured as the private key of the witness account.

//localWitnessAccountAddress =

localwitness = [
  f4df789d3210ac881cb900464dd30409453044d2777060a0c391cbdf4c6a4f57
]
```

### 活跃权限
活跃权限用于提供权限组合，例如只提供创建账户和转账功能的权限。

活跃权限具有以下功能：

1. 通过 OwnerPermission 地址可以修改 Active 权限
2. 有权限执行 AccountPermissionUpdateContract 的地址还可以修改 Active 权限
3. 支持多达 8 种组合。
4. 权限的 id 会自动从 2 开始递增。
5. 新创建账户时，会自动创建 Active 权限，并填写账户地址。默认域值为 1，密钥中只包含账户地址，权重为 1。


### 费用

1. 使用账户更新权限（即 AccountPermissionUpdate 合约）时，将收取 100TRX 的费用。
2. 使用多重签名交易，即交易中包含两个或两个以上签名的交易，除交易费外，还需支付 1TRX 的费用。
3. 上述费用可根据建议进行调整。

## API
### 修改权限
AccountPermissionUpdateContract，修改权限步骤如下：

1. 使用 getaccount 接口查询账户并获取原始权限
2. 修改权限
3. 创建合同、签名
4. 发送交易

#### http-demo

```javascript
http://{{host}}:{{port}}/wallet/accountpermissionupdate


{
  "owner_address": "41ffa9466d5bf6bb6b7e4ab6ef2b1cb9f1f41f9700",
  "owner": {
    "type": 0,
    "permission_name": "owner",
    "threshold": 2,
    "keys": [{
        "address": "41F08012B4881C320EB40B80F1228731898824E09D",
        "weight": 1
      },
      {
        "address": "41DF309FEF25B311E7895562BD9E11AAB2A58816D2",
        "weight": 1
      },
      {
        "address": "41BB7322198D273E39B940A5A4C955CB7199A0CDEE",
        "weight": 1
      }
    ]
  },
  "actives": [{
    "type": 2,
    "permission_name": "active0",
    "threshold": 3,
    "operations": "7fff1fc0037e0000000000000000000000000000000000000000000000000000",
    "keys": [{
        "address": "41F08012B4881C320EB40B80F1228731898824E09D",
        "weight": 1
      },
      {
        "address": "41DF309FEF25B311E7895562BD9E11AAB2A58816D2",
        "weight": 1
      },
      {
        "address": "41BB7322198D273E39B940A5A4C955CB7199A0CDEE",
        "weight": 1
      }
    ]
  }]
}

For the definition and limitations of the parameter fields, please see Structure Description.
```

#### 活跃权限中的操作示例
"`operations`"是一个十六进制编码序列（字节顺序为小双位），共 32 个字节（256 位），
每一位代表一种系统合同类型的权限。第 n 位表示 ID 为 n 的系统合同类型的权限，
其值为 1 表示有权执行该类型的系统合同，其值为 0 表示无权执行。不同系统合同类型的 ID 值参见下表：

| System Contract Type            | ID   | Description                                                  |
| :------------------------------ | :--- | :----------------------------------------------------------- |
| AccountCreateContract           | 0    | create Account                                               |
| TransferContract                | 1    | TRX transfer                                                 |
| TransferAssetContract           | 2    | TRC-10 token transfer                                        |
| VoteAssetContract               | 3    | unused                                                       |
| VoteWitnessContract             | 4    | Vote for Super Representatives                               |
| WitnessCreateContract           | 5    | Apply to be a Super Representative Candidate                 |
| AssetIssueContract              | 6    | Issue TRC-10 Tokens                                          |
| WitnessUpdateContract           | 8    | Update website URLs for Super Representative candidates      |
| ParticipateAssetIssueContract   | 9    | Buy TRC-10 Tokens                                            |
| AccountUpdateContract           | 10   | update account name                                          |
| FreezeBalanceContract           | 11   | Stake1.0 stake                                               |
| UnfreezeBalanceContract         | 12   | Unstake TRX staked in Stake1.0 phase                         |
| WithdrawBalanceContract         | 13   | Withdraw rewards                                             |
| UnfreezeAssetContract           | 14   | Unfreeze issued TRC10 tokens                                 |
| UpdateAssetContract             | 15   | Update TRC10 token parameters                                |
| ProposalCreateContract          | 16   | Create proposal                                              |
| ProposalApproveContract         | 17   | Approve proposal                                             |
| ProposalDeleteContract          | 18   | Delete propossal                                             |
| SetAccountIdContract            | 19   | Set account ID                                               |
| CreateSmartContract             | 30   | Create a smart contract                                      |
| TriggerSmartContract            | 31   | Trigger smart contract                                       |
| UpdateSettingContract           | 33   | Update consume_user_resource_percent                         |
| ExchangeCreateContract          | 41   | Create an exchange                                           |
| ExchangeInjectContract          | 42   | Exchange Inject                                              |
| ExchangeWithdrawContract        | 43   | Exchange Withdraw                                            |
| ExchangeTransactionContract     | 44   | Bancor Transaction                                           |
| UpdateEnergyLimitContract       | 45   | Adjust the energy limit provided by the smart contract deployer |
| AccountPermissionUpdateContract | 46   | Update account permissions                                   |
| ClearABIContract                | 48   | Clear contract ABI                                           |
| UpdateBrokerageContract         | 49   | Update SR Brokerage                                          |
| ShieldedTransferContract        | 51   | Shielded transactions                                        |
| FreezeBalanceV2Contract         | 54   | Stake TRX                                                    |
| UnfreezeBalanceV2Contract       | 55   | Unstake TRX                                                  |
| WithdrawExpireUnfreezeContract  | 56   | Withdraw the unstaked principal that has passed the lock-up period |
| DelegateResourceContract        | 57   | Resource delegate                                            |
| UnDelegateResourceContract      | 58   | Cancel resource delegate                                     |
| CancelAllUnfreezeV2Contract     | 59   | Cancel all unstakes                                          |


为方便用户阅读，以二进制大二进制字节顺序为例，
说明如何计算操作值： 位数从 0 开始，从左到右对应系统合同类型的 ID。
将二进制大二进制字节序转换为十六进制小二进制字节序，即为操作值，请参考以下示例：

| Operations Allowed                            | Binary Code(big-endian)        | Binary Code(little-endian)      | Hex Code(little-endian) |
| :-------------------------------------------- | :----------------------------- | :------------------------------ | :---------------------- |
| TransferContract(1) & VoteWitnessContract(4)  | 01001000 00000000 00000000 ... | 00010010 00000000 00000000 ...  | 12 00 00 ...            |
| TransferContract(1) & UpdateAssetContract(15) | 01000000 00000001 00000000 ... | 000000010 10000000 00000000 ... | 02 80 00 ...            |
| All system contracts                          | 11111110 11111111 11111000 ... | 01111111 11111111 00011111 ...  | 7F FF 1F ...            |


#### 主动权操作计算示例

```javascript
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import org.bouncycastle.util.encoders.Hex;

enum ContractType {
  UndefinedType(-1),
  AccountCreateContract(0),
  TransferContract(1),
  TransferAssetContract(2),
  VoteAssetContract(3),
  VoteWitnessContract(4),
  WitnessCreateContract(5),
  AssetIssueContract(6),
  WitnessUpdateContract(8),
  ParticipateAssetIssueContract(9),
  AccountUpdateContract(10),
  FreezeBalanceContract(11),
  UnfreezeBalanceContract(12),
  WithdrawBalanceContract(13),
  UnfreezeAssetContract(14),
  UpdateAssetContract(15),
  ProposalCreateContract(16),
  ProposalApproveContract(17),
  ProposalDeleteContract(18),
  SetAccountIdContract(19),
  CustomContract(20),
  CreateSmartContract(30),
  TriggerSmartContract(31),
  GetContract(32),
  UpdateSettingContract(33),
  ExchangeCreateContract(41),
  ExchangeInjectContract(42),
  ExchangeWithdrawContract(43),
  ExchangeTransactionContract(44),
  UpdateEnergyLimitContract(45),
  AccountPermissionUpdateContract(46),
  ClearABIContract(48),
  UpdateBrokerageContract(49),
  ShieldedTransferContract(51),
  MarketSellAssetContract(52),
  MarketCancelOrderContract(53),
  FreezeBalanceV2Contract(54),
  UnfreezeBalanceV2Contract(55),
  WithdrawExpireUnfreezeContract(56),
  DelegateResourceContract(57),
  UnDelegateResourceContract(58),
  CancelAllUnfreezeV2Contract(59);

  private int num;

  ContractType(int num) { this.num = num; }

  public static ContractType getContractTypeByNum(int num) {
    for(ContractType type : ContractType.values()){
      if(type.getNum() == num)
        return type;
    }
    return ContractType.UndefinedType;
  }
  public int getNum() {
    return num;
  }
}

public class  operationsEncoderAndDecoder{

  // Description: get operations code according to the input contract types
  public static String operationsEncoder(ContractType[] contractId){

    List<ContractType> list = new ArrayList<ContractType>(Arrays.asList(contractId));
    byte[] operations = new byte[32];
    list.forEach(e -> {
      int num = e.getNum();
      operations[num / 8] |= (1 << num % 8);
    });

    return Hex.toHexString(operations);
  }

  // Description: get all allowable contract types according to the operations code
  public static List<String> operationsDecoder(String operations){

    List<String> contractIDs = new ArrayList<>();
    byte[] opArray = Hex.decode(operations);
    for(int i=0;i<32;i++) // 32 bytes
    {
      for(int j=0;j<8;j++)
      {
        if((opArray[i]>>j & 0x1) ==1) {
          contractIDs.add(ContractType.getContractTypeByNum(i*8+j).name());
        }
      }
    }
    return contractIDs;
  }

  public static void main(String[] args) {
    ContractType[] contractID = {ContractType.TransferContract, ContractType.VoteWitnessContract, ContractType.FreezeBalanceV2Contract };
    String operations = operationsEncoder(contractID);
    System.out.println(operations);
    // output: 1200000000004000000000000000000000000000000000000000000000000000

    List<String> contractIDs = operationsDecoder(operations);
    contractIDs.forEach(e ->{
      System.out.print(e + " ");
    });
    // output: TransferContract VoteWitnessContract FreezeBalanceV2Contract
  }
}
```

## 构建并执行多重签名交易

1. 创建一个事务，与非多重签名事务的创建过程相同
2. 指定 Permission_id，默认为 0，表示所有者权限
3. 用户 A 通过其他方式向 B 签署后交易。
4. 用户 B 签名，签名后的交易通过其他方式发送给 C。
    - n. 最后完成签名的用户向节点广播交易。
    - N+1，验证多重签名的权重之和大于域值，接受交易，否则拒绝交易


### 其他多重签名相关接口
查询与多重签署交易相关的 API：

#### 1.查询签名地址
```shell
curl -X POST  http://127.0.0.1:8090/wallet/getapprovedlist -d '{"transaction"}'
 
rpc GetTransactionApprovedList(Transaction) returns (TransactionApprovedList) { }
```

#### 2.查询交易签名权重

```shell
curl -X POST  http://127.0.0.1:8090/wallet/getsignweight -d '{"transaction"}'
 
rpc GetTransactionSignWeight (Transaction) returns (TransactionSignWeight) {}
```

所有者权限和活动权限是在帐户创建期间自动生成的。 Owner-permission 包含一个 key，权限和阈值均设置为 1。
active-permission 还包含一个 key，权限和阈值均设置为 1。
操作为"`7fff1fc0033efb07000000000000000000000000000000000000000000000000`"，即支持除 AccountPermissionUpdateContract 之外的所有操作。






































