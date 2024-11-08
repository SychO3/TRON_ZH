# 在波场网络上质押
***
波场网络拥有三种系统资源：

- 能量
- 带宽
- 投票权

## 如何质押以获取系统资源
账户持有者通过质押获取能量和带宽资源。
请参考 `wallet/freezebalancev2` 以了解如何通过 HTTP API 完成质押操作，
并参考 `Stake2.0 Solidity API` 以了解如何通过合约完成质押操作。

波场通过质押机制分配资源。质押 TRX 不仅可以获取带宽或能量资源，
还可以获得与质押数量相等的投票权（波场权力，简称 TP）。质押 1TRX，将获得 1TP。
质押获得的能量或带宽资源用于支付交易费用，获得的投票权用于投票选举超级代表以获取投票奖励。

解锁操作将释放相应的资源。

## 如何代理资源
账户通过质押获取能量或带宽资源后，可以通过代理操作将资源代理到其他地址，
也可以通过取消代理操作收回已分配的资源。代理资源时请注意以下情形：

- 只有能量和带宽可以代理到其他地址，投票权不可代理。
- 只有通过 `Stake2.0` 质押获得的未使用资源可以代理到其他地址。
- 能量/带宽只能代理到已激活的外部账户地址，而不能代理到合约地址。

您可以使用 `wallet/getcandelegatedmaxsize` 接口查询账户中某种资源类型可用的代理份额。
代理资源时可以使用时间锁和锁定周期。如果时间锁为 `true`，资源代理完成后，
只有在锁定周期 lock_period 指定的时间过去后，才能取消对该地址的资源代理。
在锁定周期中，如果用户再次对相同地址进行相同类型的资源代理，锁定时间将被设置为新设置的值。
如果不使用时间锁，资源代理完成后可以立即取消代理。

## 如何解锁 TRX
完成 TRX 质押后，您可以随时解锁。解锁后，
您需要等待14天才能将解锁的 TRX 提取到您的账户中。
14天是波场网络的第70号参数，可以通过网络治理提案进行投票。
请参阅 `unfreezebalancev2` 以了解如何通过 HTTP API 完成解锁余额，
并使用 `withdrawexpireunfreeze` 将已过14天的解锁 TRX 提取到您的账户中。

质押的 TRX 可以多次部分解锁，但最多同时允许32次解锁操作。
也就是说，当用户发起第一次解锁操作时，在第一次解锁的 TRX 到达并准备好提取到其账户之前，
他或她只能再发起31次解锁操作。可以通过 `getavailableunfreezecount` 接口查询剩余的解锁次数。

已代理的 TRX 不可解锁。除了失去相同数量的资源份额外，解锁还会失去相同数量的 TP 资源。

解锁时，如果有先前解锁的 TRX 已过锁定期，则此次解锁操作会将已过锁定期的解锁 TRX 同时提取到账户中。
您可以使用 `gettransactioninfobyid` API 在 `withdraw_expire_amount` 字段中查询已过锁定期的解锁 TRX 的提取数量。

### TP 回收
在解锁 Stake2.0 阶段质押的 TRX 后，将失去相同数量的投票权。
系统将优先回收账户中的闲置投票权。如果闲置的 TP 不足，将继续回收已用的 TP。
如果用户为多个超级代表投票，将按比例从每个超级代表中撤回一定数量的票数，并回收相应的投票权。每个超级代表撤回票数的计算公式为：

!!!note ""
    当前超级代表的撤回票数 = 总撤回票数 * (当前超级代表的票数 / 该账户的总票数)

例如，假设用户 A 质押了 2000TRX 并获得 2000TP，
其中 1000TP 投给了2个超级代表，分别为 600 票和 400 票，另有 1000TP 保留在账户中。
此时，A 解锁 1500TRX，即需要从 A 的账户中回收 1500TP。
在这种情况下，A 账户中的闲置 1000TP 将首先被撤回，剩余的 500TP 将从已投票的 TP 中撤回，
分别从两个超级代表中撤回 300 和 200 TP。投票的计算方法如下：

!!!note ""
    超级代表 1 撤回的票数 = 500 * (600 / 1 000) = 300
    超级代表 2 撤回的票数 = 500 * (400 / 1,000) = 200

目前，波场网络使用 Stake2.0 质押机制，但通过 Stake1.0 获得的资源和投票仍然有效。
Stake1.0 质押的 TRX 仍可通过 Stake1.0 API 提取，
但需注意，如果解锁 Stake1.0 中质押的 TRX，账户中的所有投票将被撤销。

## 如何取消解锁操作
在用户解锁 TRX 后，Stake2.0 支持取消所有解锁操作。
这样可以再次将资产用于质押以获得相应资源，而不必等待解锁资金经过锁定期后才能将资金提取到账户，
然后再次质押。请参考 `cancelallunfreezev2` 以了解如何通过 HTTP API 取消所有解锁操作。

在取消解锁操作时，所有仍在等待期内的解锁资金将被重新质押，并且通过重新质押获得的资源保持不变。
超过14天等待期的解锁操作无法取消，此部分解锁 TRX 将自动提取到所有者账户。
用户可以通过 `gettransactioninfobyid` 接口查询被取消的解锁 TRX 数量 `cancel_unfreezeV2_amount`，以及已超过锁定期的提取 TRX 数量 `withdraw_expire_amount`。

## API
权益模型的相关接口及其说明如下表：

| 接口名称               | 描述                                 |
|--------------------|------------------------------------|
| freezebalancev2    | 质押 TRX                             |
| unfreezebalancev2  | 解锁 TRX                             |
| unfreezebalance    | 解锁 Stake1.0 期间质押的 TRX              |
| delegateresource   | 代理资源                               |
| undelegateresource | 取消代理资源                             |
| withdrawexpireunfreeze | 提取未冻结 TRX                          |
| getavailableunfreezecount	| 查询剩余的解锁次数                          |
| getcanwithdrawunfreezeamount	| 查询可提取 TRX                          |
| getcandelegatedmaxsize	| 查询账户中某种资源类型可用的代理份额                 |
| getdelegatedresourcev2	| 查询由 fromAddress 委托给 toAddress 的资源量 |
| getdelegatedresourceaccountindexv2    | 查询账户资源委托索引                         |
| getaccount	| 查询账户质押状态、资源份额、解锁状态、投票状态            |
| getaccountresource	| 查询资源总量、已使用量、可用量                    |
| cancelallunfreezev2	| 取消解锁                               |