# 在波场网络上质押
***
波场网络拥有三种系统资源：

- 能量
- 带宽
- 投票权

## 如何质押获取系统资源
能量和带宽资源由账户所有者通过质押获得，
请参考 `wallet/freezebalancev2` 了解如何通过 HTTP API 完成质押操作，
请参阅 `Stake2.0 Solidity API` 了解如何通过合约完成质押操作。

TRON通过Stake机制来分配资源。
质押TRX除了获得带宽或能源资源外，
还将获得与质押金额等值的投票权（TRON Power，简称TP）。
质押1 TRX，您将获得1TP。通过质押获得的能量或带宽资源用于支付交易费用，
获得的投票权用于投票给超级代表以获得投票奖励。

解押操作会释放相应的资源。

## 如何委派资源
账户通过质押获得能量或带宽资源后，可以通过委托操作将资源委托给其他地址，
也可以通过取消委托操作收回分配的资源。资源委托时请注意以下情况：

- 只能委托能量和带宽给其他地址，不能委托投票权
- 只有通过Stake2.0质押获得的未使用资源才能委托给其他地址
- 能量/带宽只能委托给激活的外部账户地址，不能委托给合约地址

您可以使用wallet/getcandelegatemaxsize接口查询账户中某种资源类型的可用委托份额。
委派资源时可以使用 timelock 和 lock_period。
如果时间锁为true，则资源委托完成后，
只有经过lock_period指定的锁定周期后才能取消该地址的资源委托。
锁定期间，如果用户再次对同一地址进行同类型资源委托，则锁定时间将设置为新设置的值。
如果不使用时间锁，资源委托后可以立即取消委托。

## 如何取消质押TRX
完成TRX质押后，您可以随时解押。解押后，您需要等待14天才能将未质押的TRX提现至您的账户。 
14天是TRON网络的第70个参数，可以通过网络治理提案进行投票。
请参考unfreezebalancev2了解如何通过HTTP API完成余额解冻，
并使用withdrawexpireunfreeze将已超过14天未质押的TRX提取到您的账户中。

可以多次部分解除对 TRX 的锁定，但最多只能同时进行 32 次解除锁定操作。
也就是说，当用户发起第一次解冻操作时，在第一次解冻的 TRX 到达并准备提取到用户账户之前，
用户只能再发起 31 次解冻操作。剩余的解冻次数可通过 getavailableunfreezecount 接口查询。

已委托的 TRX 不能解除委托。除了失去相同数量的资源共享外，取消委托还将失去相同数量的 TP 资源。

在解除锁定时，如果先前有一个已过锁定期的未锁定本金，
则该解除锁定操作将同时向账户提取已过锁定期的未锁定本金。
您可以使用 gettransactioninfobyid API 在 withdraw_expire_amount 字段中查询已过锁定期的未锁定 TRX 的提取金额。

### TRON 电力回收
在解除 Stake2.0 阶段所投入的 TRX 后，将失去相同数量的投票权。
系统将首先收回账户中的闲置投票权。如果闲置 TP 不足，系统将继续收回已使用的 TP。
如果用户为多个超级代表投票，则会按比例从每个超级代表中收回一定数量的投票，并收回相应的投票权。
每个 SR 的撤票计算公式为：

!!!note ""
    从现任超级代表中撤回的票数 = 拟撤回的总票数 * （现任超级代表的票数/该账户的总票数）

例如，假设 A 下注了 2,000TRX 并获得 2,000 TRON Power，
其中 1,000 TRON Power 分别投给了 2 个超级代表、600 票和 400 票，
1,000 TRON Power 仍留在账户中。此时，A 解押了 1 500TRX，
这意味着需要从 A 的账户中收回 1 500 TRON Power。
在这种情况下，A 账户中闲置的 1,000 TP 将被先行提取，
而幸免于难的 500 TP 将从已投票的 TP 中提取，
即从两个超级代表的账户中分别提取 300 TP 和 200 TP。以下是票数的计算方法：

!!!note ""
    超级代表 1 撤回的票数 = 500 * (600 / 1 000) = 300
    超级代表 2 撤回的票数 = 500 * (400 / 1,000) = 200

目前，TRON 网络采用 Stake2.0 股权机制，
但 Stake1.0 获得的资源和票数仍然有效。
通过 Stake1.0 API 提取在 Stake1.0 入股的 TRX 仍然有效，
但需要注意的是，如果在 Stake1.0 入股的 TRX 被取消入股，账户中的所有票数都将被取消。

## 如何取消质押
Stake2.0支持用户解押TRX后取消所有解押，使资产再次用于质押以获得相应的资源，
无需等待未质押的资金过了锁定期才将资金提现到账户，然后再次放样它们。
请参考cancelallunfreezev2了解如何通过HTTP API取消所有质押操作。

取消质押时，所有处于等待期内的未质押资金将被重新质押，
通过重新质押获得的资源与之前相同。超过14天等待期的质押无法取消，
这部分质押资金将自动提取至所有者账户。
用户可以通过gettransactioninfobyid接口查询已取消的未质押本金金额cancel_unfreezeV2_amount、
以及超过锁仓期的提取本金金额withdraw_expire_amount。

## API
权益模型的相关接口及其说明如下表：

| 接口名称               | 描述 |
|--------------------| --- |
| freezebalancev2    | 质押TRX |
| unfreezebalancev2  | 解除质押TRX |
| unfreezebalance    | 取消 Stake1.0 期间质押的 TRX |
| delegateresource   | 委托资源 |
| undelegateresource | 取消委托资源 |
| withdrawexpireunfreeze | 提取未冻结余额 |
| getavailableunfreezecount	| 查询执行取消抵押操作的剩余次数 |
| getcanwithdrawunfreezeamount	| 查询可提现余额 |
| getcandelegatedmaxsize	| 查询指定资源类型的可委托资源共享量 |
| getdelegatedresourcev2	| 查询由 fromAddress 委托给 toAddress 的资源量 |
| getdelegatedresourceaccountindexv2    | 查询账户资源委托索引 |
| getaccount	| 查询账户质押状态、资源份额、解押状态、投票状态 |
| getaccountresource	| 查询资源总量、已使用量、可用量 |
| cancelallunfreezev2	| 取消质押 |