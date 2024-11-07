# 多重签名示例流程
***
开发人员可以使用 SDK 轻松发送多重签名交易。
本文将以账户 `TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ` 为例，介绍如何使用 tronweb SDK 在 Nile 测试网络上创建多签名转账交易。

多重签名交易的步骤如下：

1. 修改账户权限
2. 查询账户权限
3. 选择权限并创建交易
4. 执行多重签名
5. 检查交易的签署权重
6. 检查批准列表
7. 广播交易

## 1. 修改账户权限
通过 tronscan 或使用 `wallet/accountpermissionupdate` API 修改帐户权限。在此示例中，
名为`NewAddedActivePermission`的新活动权限通过 tronscan 添加到帐户 `TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ`。


## 2. 查询账户权限
使用 `wallet/getaccount` API 查询账户权限信息：

```shell
curl --location --request POST 'https://api.nileex.io/wallet/getaccount' \
--header 'Content-Type: text/plain' \
--data-raw '{
     "address": "TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ",
     "visible": true
}'
```       

返回：

```javascript
{
    "address": "TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ",
    "balance": 2897800000,
    ......
    "owner_permission": {
        "permission_name": "owner",
        "threshold": 1,
        "keys": [
            {
                "address": "TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ",
                "weight": 1
            }
        ]
    },
    "active_permission": [
        {
            "type": "Active",
            "id": 2,
            "permission_name": "active",
            "threshold": 1,
            "operations": "7fff1fc0033efb0f000000000000000000000000000000000000000000000000",
            "keys": [
                {
                    "address": "TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ",
                    "weight": 1
                }
            ]
        },
        {
            "type": "Active",
            "id": 3,
            "permission_name": "NewAddedActivePermission",
            "threshold": 2,
            "operations": "77ff07c00260c30f000000000000000000000000000000000000000000000000",
            "keys": [
                {
                    "address": "TXTMqofe9nS5bN5tfhfd6ayWocJm7oxJKT",
                    "weight": 1
                },
                {
                    "address": "TVqTEPUPiTxhzaSnD9xXEvarUQooLibkXM",
                    "weight": 1
                }
            ]
        }
    ],
    ......
}
```

从返回的结果可以看出，账户 `TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ` 拥有活跃权限，id 为 3，阈值为 2，授权给两个账户，每个账户的权重为 1。

## 3. 选择权限并创建交易

在本例中，我们选择使用 `id` 为 `TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ` 的活动权限 `3`，通过 `tronWeb.transactionBuilder.sendTrx` 方法构建多签名TRX转账交易：

```javascript
var unsignedTransaction = await tronWeb.transactionBuilder.sendTrx('TVqTEPUPiTxhzaSnD9xXEvarUQooLibkXM', 10000000, 'TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ',{permissionId: 3});
```

- 发送方：交易中的所有者地址，即 `TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ`。在建立交易时，无需关心交易创建者，只需将交易中 TRX 的发送方设置为多重签名地址即可。
- 多重签名权限： 选择 `TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ` - `{permissionId： 3}`

## 4. 执行多重签名
`id` 为 `3` 的 `TUZKijZ9Esy8JEkrqMpaVgtbDKKNA5p5CZ` 的 Active 权限已授权给 `TXTMqofe9nS5bN5tfhfd6ayWocJm7oxJKT` 和 `TVqTEPUPiTxhzaSnD9xXEvarUQooLibkXM`。
这两个账户可以使用 `tronWeb.trx.multiSign` API 签署交易：

```javascript
var signedTransaction = await tronWeb.trx.multiSign(unsignedTransaction, 'f0bd085afbcf31374cf6ae4585faee1366f8c850c596e2649ba93015ac479f74');

signedTransaction = await tronWeb.trx.multiSign(signedTransaction, '6c72f51dc78d24ce1517912526a8a0e73379694ae7594efd3e05adb33d726edc');
```

返回：

```javascript
{
  visible: false,
  txID: '507efa43ef267005cd384f74f8bc2b2d6e807807144144fd01f194f36ddfb93a',
  raw_data: {
    contract: [ [Object] ],
    ref_block_bytes: 'e5b2',
    ref_block_hash: 'c565570704ef1e7a',
    expiration: 1700538150000,
    timestamp: 1700538091138
  },
  raw_data_hex: '0a02e5b22208c565570704ef1e7a40f0d8e3ffbe315a6a080112640a2d747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5472616e73666572436f6e747261637412330a1541cbe603e1f0ac26c50bb795d8d54a95706c64b2e2121541d9eb1101ba37f55c436cc668ed1ac18d23d6c6631880ade204280370828de0ffbe31',
  signature: [
    '9f51d0839b8dd10b5a6bc3b043f6246d5c8a00a79839a52b0987ee337911b3375d0376c3b1aef3a7ffa0779a2e3d0cb95380294e6b934f738964a0551675d45501',
    '2bc656a647cbd9de932bf63909f73aacf2e97f0c771bd72b69654f6242a152e8ecbc8712ad59fe95a280cffbed5ef2dd28777dc27267976fe0c6374e00ba355c01'
  ]
}

```

## 5. 检查交易的签署权重

通过 `tronweb.trx.getSignWeight` 方法，不仅可以检查权限要求的权重，还可以检查当前有多少地址已签名以及当前权重。
该方法可在多重签名过程中或之后调用。

```javascript
var signWeight = await tronWeb.trx.getSignWeight(signedTransaction);
```

返回：

```javascript
{
  result: {},
  approved_list: [
    '41d9eb1101ba37f55c436cc668ed1ac18d23d6c663',
    '41ebadab040181bcc649169d00c28a3ad1bb6bfeb7'
  ],
  permission: {
    operations: '77ff07c00260c30f000000000000000000000000000000000000000000000000',
    keys: [ [Object], [Object] ],
    threshold: 2,
    id: 3,
    type: 'Active',
    permission_name: 'NewAddedActivePermission'
  },
  current_weight: 2,
  transaction: {
    result: { result: true },
    txid: '507efa43ef267005cd384f74f8bc2b2d6e807807144144fd01f194f36ddfb93a',
    transaction: {
      signature: [Array],
      txID: '507efa43ef267005cd384f74f8bc2b2d6e807807144144fd01f194f36ddfb93a',
      raw_data: [Object],
      raw_data_hex: '0a02e5b22208c565570704ef1e7a40f0d8e3ffbe315a6a080112640a2d747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5472616e73666572436f6e747261637412330a1541cbe603e1f0ac26c50bb795d8d54a95706c64b2e2121541d9eb1101ba37f55c436cc668ed1ac18d23d6c6631880ade204280370828de0ffbe31'
    }
  }
}
```

## 6. 检查批准列表
您还可以通过 `tronWeb.trx.getApprovedList` 查看已签署交易的账户列表。

```javascript
var approvedList = await tronWeb.trx.getApprovedList(signedTransaction);
```

返回：

```javascript
{
  result: {},
  approved_list: [
    '41d9eb1101ba37f55c436cc668ed1ac18d23d6c663',
    '41ebadab040181bcc649169d00c28a3ad1bb6bfeb7'
  ],
  transaction: {
    result: { result: true },
    txid: '507efa43ef267005cd384f74f8bc2b2d6e807807144144fd01f194f36ddfb93a',
    transaction: {
      signature: [Array],
      txID: '507efa43ef267005cd384f74f8bc2b2d6e807807144144fd01f194f36ddfb93a',
      raw_data: [Object],
      raw_data_hex: '0a02e5b22208c565570704ef1e7a40f0d8e3ffbe315a6a080112640a2d747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5472616e73666572436f6e747261637412330a1541cbe603e1f0ac26c50bb795d8d54a95706c64b2e2121541d9eb1101ba37f55c436cc668ed1ac18d23d6c6631880ade204280370828de0ffbe31'
    }
  }
}
```


## 7. 广播交易
多重签名完成后，可以直接广播已签名的事务，并在稍后使用 `getTransactionById` 检查事务。

```javascript
var result = await tronWeb.trx.broadcast(signedTransaction);
```

返回：
```javascript
{
  result: true,
  txid: '507efa43ef267005cd384f74f8bc2b2d6e807807144144fd01f194f36ddfb93a',
  transaction: {
    visible: false,
    txID: '507efa43ef267005cd384f74f8bc2b2d6e807807144144fd01f194f36ddfb93a',
    raw_data: {
      contract: [Array],
      ref_block_bytes: 'e5b2',
      ref_block_hash: 'c565570704ef1e7a',
      expiration: 1700538150000,
      timestamp: 1700538091138
    },
    raw_data_hex: '0a02e5b22208c565570704ef1e7a40f0d8e3ffbe315a6a080112640a2d747970652e676f6f676c65617069732e636f6d2f70726f746f636f6c2e5472616e73666572436f6e747261637412330a1541cbe603e1f0ac26c50bb795d8d54a95706c64b2e2121541d9eb1101ba37f55c436cc668ed1ac18d23d6c6631880ade204280370828de0ffbe31',
    signature: [
      '9f51d0839b8dd10b5a6bc3b043f6246d5c8a00a79839a52b0987ee337911b3375d0376c3b1aef3a7ffa0779a2e3d0cb95380294e6b934f738964a0551675d45501',
      '2bc656a647cbd9de932bf63909f73aacf2e97f0c771bd72b69654f6242a152e8ecbc8712ad59fe95a280cffbed5ef2dd28777dc27267976fe0c6374e00ba355c01'
    ]
  }
}

```
