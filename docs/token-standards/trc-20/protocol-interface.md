# 协议接口
***
## TRC-20 合约标准
TRC-20 是一组用于发行通证资产的合约标准，符合该标准编写的合约被视为 TRC-20 合约。
当钱包和交易所对接 TRC-20 合约的资产时，可以从这套标准中知道合约定义了哪些功能和事件，以便于对接。

### 可选项

!!! quote "通证名称"
    string public name = "TRONEuropeRewardCoin";


!!! quote "通证缩写"
    string public symbol = "TERC";


!!! quote "通证精度（小数位）"
    uint8 public decimals = 6;


### 必需项

```javascript
contract TRC20 {
             function totalSupply() constant returns (uint theTotalSupply);
             function balanceOf(address _owner) constant returns (uint balance);
             function transfer(address _to, uint _value) returns (bool success);
             function transferFrom(address _from, address _to, uint _value) returns (bool success);
             function approve(address _spender, uint _value) returns (bool success);
             function allowance(address _owner, address _spender) constant returns (uint remaining);
             event Transfer(address indexed _from, address indexed _to, uint _value);
             event Approval(address indexed _owner, address indexed _spender, uint _value);
}
```

`totalSupply()`

此函数返回通证的总供应量。

`balanceOf()`

此函数返回特定账户的通证余额。

`transfer()`

此函数用于将一定数量的通证转移到特定地址。

`approve()`

此函数用于授权第三方（如 DAPP 智能合约）从通证持有者的账户中转移通证。

`transferFrom()`

此函数用于允许第三方将通证从持有者账户转移到接收者账户。持有者账户必须被授权可被第三方调用。

`allowance()`

此函数用于查询第三方可以转移的剩余通证数量。


### 事件函数

当通证成功转移时，合约将触发一个转移事件。

!!! quote ""
    event Transfer(address indexed _from, address indexed _to, uint256 _value)

当 `approve()` 成功调用时，合约将触发一个授权事件。

!!! quote ""
    event Approval(address indexed _owner, address indexed _spender, uint256 _value)





