# 获取 TRC-20 交易历史记录

***

获取特定账户中某个 TRC-20 的交易历史。


参数说明：

- version：最新版本 v1。
- address：账户地址，以 Base58 或 Hex 格式表示。
- only_confirmed：true|false。如果为 false，则返回已确认和未确认的交易；如果没有参数，则返回已确认和未确认的交易。不能与 only_unconfirmed 一起使用。
- only_unconfirmed：true|false。如果为 false，则返回已确认和未确认的交易；如果没有参数，则返回已确认和未确认的交易。不能与 only_confirmed 一起使用。
- limit：每页的交易数量，默认是20，最大为200。
- fingerprint：上一页返回的最后一笔交易的指纹。使用此参数时，其他参数和过滤条件应保持不变。
- contract_address：TRC20 合约地址，以 Base58 或 Hex 格式表示。



```shell
// 示例
// 获取地址 TJmmqjb1DK9TTZbQXzRQ2AuA94z4gKAPFh 上与 TRC20 USDT 相关的交易
curl --request GET \
  --url 'https://api.trongrid.io/v1/accounts/TJmmqjb1DK9TTZbQXzRQ2AuA94z4gKAPFh/transactions/trc20?limit=20&contract_address=TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t'
```