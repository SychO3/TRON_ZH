# 资源模型
***
投票权、带宽和能量是 TRON 网络的重要系统资源。
其中，投票权用于投票选举超级代表；带宽是衡量区块链数据库中存储的交易字节大小的单位。
交易越大，消耗的带宽资源就越多。能量是衡量 TRON 虚拟机在 TRON 网络上执行特定操作所需的计算量的单位。
由于智能合约交易的执行需要计算资源，因此每笔智能合约交易都需要支付能量费用。

## 投票权
任何账户在投票选举超级代表之前，都需要获得投票权，即 TRON 权力（TP）。
投票权可通过质押 TRX 获得。除了获得带宽或能量外，质押 TRX 还可同时获得投票权。
质押 1TRX 的投票者将获得 1TP。有关如何质押，请参阅 TRON 网络上的质押章节。

投票者可以多次质押，多次质押获得的投票权将添加到投票者的账户中。
投票者可以通过`wallet/getaccountresource`接口查询账户拥有的投票权总数以及已使用的投票权数量。

## 带宽
所有类型的交易都需要消耗带宽（Bandwidth Points）。
交易以字节数组的形式在 TRON 网络中传输和存储。
一个字节需要一个带宽，因此事务需要消耗的带宽等于事务字节数。

当可用带宽不足时，需要燃烧 TRX 以支付带宽：

!!! note ""
    消耗的 TRX = 消耗的带宽量 * 带宽单价

目前，带宽单价为1000sun。

### 如何获取带宽
每个外部账户每天有600个免费带宽，通过质押TRX可以获得更多带宽。
所有用户根据质押的TRX数量共享固定数量的带宽。全网固定带宽供应总量为432亿/天。
可以通过以下公式计算质押一定数量的TRX可以获得多少带宽：

!!! note ""
    获取带宽量 = 获取带宽质押的TRX数量 / 全网获取带宽质押的TRX总量 * 43_200_000_000

您可以通过`wallet/getaccountresource`接口获取为获取全网带宽而质押的TRX总量。

您可以发送 `FreezeBalanceV2Contract` 类型的交易来质押 TRX 以获得带宽。
下面以`wallet-cli为例创建FreezeBalanceContract`类型的交易：

!!! note ""
    wallet> freezeBalanceV2 1000000 0

### 带宽消耗
除了查询操作外，任何事务都需要消耗带宽。
带宽消耗规则为：首先检查交易发起者质押TRX获得的带宽是否充足，
如果充足则消耗质押TRX获得的带宽；否则，检查交易发起方的免费带宽是否充足，
如果充足，则消耗免费带宽，否则将按照每带宽0.001TRX的单价燃烧TRX来支付交易的带宽。

### 带宽恢复
账户的免费带宽和质押TRX获得的带宽消耗完后，会在24小时内逐渐恢复。

### 带宽余额查询
首先调用节点HTTP接口`wallet/getaccountresource`获取账户当前资源状态，然后通过以下公式计算带宽余额：

!!! note ""
    Free bandwidth balance = freeNetLimit - freeNetUsed

    Bandwidth balance obtained by staking TRX = NetLimit - NetUsed

!!! tip
    如果接口返回的结果不包含上述公式中的参数，则表示参数值为 0。

## 能量
智能合约每条指令的执行在运行时都会消耗一定的能量。
因此不同复杂度的合约消耗不同数量的能量。
合约执行时，按照指令一一计算并扣除能量。
当账户可用能量不足时，需要燃烧TRX来支付相应的能量。

!!! note ""
    Burned TRX = Energy quantity * the unit price of Energy

目前，能量的单价为210sun。

### 如何获取能量
能量只能通过质押TRX来获得。
所有用户根据质押的TRX数量共享固定数量的能量。
全网每日固定总能源供应量为180,000,000,000。
请使用以下公式计算质押一定数量的TRX可以获得多少能量：

!!! note ""
    获取能量数量 = 获取能量质押的TRX数量 / 全网获取能量质押的TRX总量 * 180_000_000_000

您可以通过`wallet/getaccountresource`接口获取全网获取能量所质押的TRX总量。

您可以发送 `FreezeBalanceV2Contract` 类型的交易来质押 TRX 以获得能量。
下面以wallet-cli为例创建`FreezeBalanceContract`类型的交易：

!!! note ""
    wallet> freezeBalanceV2 1000000 1

### 能量消耗
合约执行时，按照指令一一计算并扣除能量。账户能耗优先级如下：

- 质押TRX获得的能量
- 燃烧TRX

首先，质押TRX获得的能量会被消耗。如果这部分能量不够，
账户的TRX将继续被燃烧，以支付交易所需的能量资源，按照每能量0.00021TRX的单价。

如果合约在执行过程中因抛出revert异常而退出，则仅扣除已经执行的指令所消耗的能量。
但对于异常合约，例如合约执行超时，或者由于bug导致异常退出，该笔交易的最大可用能量将会被扣除。
您可以通过设置交易的`fee_limit`参数来限制本次交易的最大能源成本。

### 能量恢复
账户能量资源消耗后，会在24小时内逐渐恢复。

### 能量余额查询
首先调用节点HTTP接口`wallet/getaccountresource`获取账户当前的资源状态，
然后通过以下公式计算能量余额：

!!! note ""
    Energy Balance = EnergyLimit - EnergyUsed

!!! tip
    如果接口返回的结果不包含上述公式中的参数，则表示参数值为 0。


## 动态能量模型
动态能源模型是TRON网络的一种资源平衡机制，可以根据合约的资源占用情况，
动态调整各合约的能源消耗，从而使链上能源资源的分配更加合理，
防止过度使用。将网络资源集中在少数热门合约上。详细内容请参见[动态能量模型简介](https://coredevs.medium.com/introduction-to-dynamic-energy-model-31917419b61a)。

### 原理
如果某个合约在一个维护周期内使用了过多的资源，那么在下一个维护周期内，
将增加一定比例的惩罚性消耗，用户向该合约发送相同的事务将比之前消耗更多的能量。
当合约合理使用资源后，调用该合约的用户产生的能耗将逐渐恢复正常。

每个合约都有一个 energy_factor 字段，
表示智能合约交易的能耗相对于基础能耗的增加比例，初始值为 0。
当合约的 energy_factor 为 0 时，表示合约正在合理使用资源，
调用该合约不会有额外的能耗。当 energy_factor 大于 0 时，表示该合约已成为热门合约，
调用该合约时会消耗额外的能量。可以通过 getcontractinfo API 查询合约的 energy_factor。

合约调用交易最终消耗的能量计算公式如下：

!!! note ""
    合约调用交易的能量 = 交易产生的基本能量 * （1 + 能量系数）

动态能量模型引入了TRON网络的以下三个参数，共同控制合约的energy_factor字段：

- 阈值：合同基本能耗阈值。在一个维护周期内，如果合同的基本能耗超过此阈值，下一个维护周期的合同能耗将增加。
- increase_factor：如果在某个维护周期内，合同的基本能耗超过了阈值，则在下一个维护周期内，energy_factor 将根据 increase_factor 增加一定的百分比。
- max_factor：能量系数的最大值。

还有一个变量 decrease_factor，用于降低合同的能量系数：

- decrease_factor（减少系数）： increase_factor 的 1/4。当合同的基本能耗低于阈值时，energy_factor 将根据 decrease_factor 降低一定的百分比。

当合同的基本能耗在一个维护周期内超过阈值时，其能源系数将在下一个维护周期内增加，但最大值不会超过 max_factor，计算公式为： 当合同的基本能耗在一个维护周期内超过阈值时，其能源系数将在下一个维护周期内增加，但最大值不会超过 max_factor：

!!! note ""
    energy_factor = min((1 + energy_factor) * (1 + increaese_factor)-1, max_factor)

当合同的基本能耗在一个维护周期内降至阈值以下时，下一个维护周期的能源系数将下降，但最小值不会低于 0。 计算公式如下：

!!!note ""
    energy_factor = max((1 + energy_factor) * (1 - decrease_factor)- 1, 0)

主网络已启用动态能源模型，相关参数设置如下：

- threshold：5,000,000,000
- increase_factor：0.2
- max_factor：3.4

由于流行合约在不同维护周期的能耗不同，因此在调用合约时有必要为事务设置适当的 feelimit 参数。更多信息，请参阅设置事务 feelimit。

### API
下表列出了动态能源模型的相关界面及其说明：

| 接口名称                        | 描述               | 返回值与动态能量模型相关 |
|-----------------------------|------------------|----------------------------------------------|
| `getcontractinfo`           | 查询合同信息           | contract_state.energy_usage：表示当前维护周期内合同的基本能源使用总量； contract_state.energy_factor：合同的能源系数，0 表示非热门合同，大于 0 表示热门合同； contract_state.update_cycle：当前维护周期的编号 |
| `triggerconstantcontract`   | 查询合同数据或估算能耗      | energy_penalty 表示罚则能量；energy_used 表示总能量（基本能量和罚则能量之和）。|
| `gettransactioninfobyid`    | 查询交易信息	          | receipt.energy_penalty_total 表示罚则能量 |
 | `gettransactionreceiptbyid` | 查询交易执行结果、费用和其他信息 | receipt.energy_penalty_total 表示罚则能量 |


