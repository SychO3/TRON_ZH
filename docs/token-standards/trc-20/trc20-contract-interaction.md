# TRC-20 合约交互

***

以 Shasta 测试网的 USDT 合约为例，分别使用 Tronweb 和 wallet-cli 调用合约的 TRC-20 接口。

一些相关链接：

- [在 Tronscan 上查找 USDT](https://shasta.tronscan.org/#/contract/TQQg4EL8o1BSeKJY4MJ8TB8XK7xufxFBvK/code)
- [代码转换工具](https://shasta.tronscan.org/#/tools/tron-convert-tool)

我们可以使用 `triggersmartcontract` 函数调用合约中的常量函数，以便直接获取结果而无需广播。请在节点配置中设置 `supportConstant = true`。

## 常量调用
常量包括了 name、symbol、decimals、totalSupply 等函数，这些函数不会修改区块链状态，也不会消耗能量。

=== "HTTP API"

    ``` shell
        /wallet/triggerconstantcontract` 
        描述：触发智能合约的常量，交易不在区块链上  
        示例：  
        curl -X POST https://127.0.0.1:8090/wallet/triggerconstantcontract -d '{
        "contract_address":"419E62BE7F4F103C36507CB2A753418791B1CDC182",
        "function_selector":"name()", # 可以是 name()、symbol()、decimals()、totalSupply() 等
        "owner_address":"41977C20977F412C2A1AA4EF3D49FEE5EC4C31CDFB"
        }'
    ```

=== "Tronweb"

    ``` javascript
        const TronWeb = require('tronweb')
        
        const HttpProvider = TronWeb.providers.HttpProvider;
        const fullNode = new HttpProvider("https://127.0.0.1:8090");
        const solidityNode = new HttpProvider("https://127.0.0.1:8090");
        const eventServer = new HttpProvider("https://127.0.0.1:8090");
        const privateKey = "your private key";
        const tronWeb = new TronWeb(fullNode,solidityNode,eventServer,privateKey);
        
        async function triggerSmartContract() {
            const trc20ContractAddress = "TQQg4EL8o1BSeKJY4MJ8TB8XK7xufxFBvK";//合约地址
        
            try {
                let contract = await tronWeb.contract().at(trc20ContractAddress);
                //使用 call 执行纯粹或查看智能合约方法。
                //这些方法不会修改区块链，执行不花费任何费用，并且也不广播到网络。
                let result = await contract.name().call(); // 可以是 name()、symbol()、decimals()、totalSupply() 等
                console.log('result: ', result);
            } catch(error) {
                console.error("触发智能合约错误",error)
            }
        }
    ```

=== "Wallet-cli"

    ``` shell
        TriggerConstantContract TQQg4EL8o1BSeKJY4MJ8TB8XK7xufxFBvK name() # false
        # 用法：TriggerConstantContract [ownerAddress] [contractAddress] [method] [args] [isHex]
        # 参数描述：  
        # ownerAddress: 调用者地址  
        # contractAddress：TRC20 合约地址  
        # method：合约函数，可以是 name()、symbol()、decimals()、totalSupply() 等
        # args：函数参数，如果没有参数，使用 `#` 占位符  
        # isHex：命令参数的地址是否为十六进制格式
    ```

## 余额查询


## 转账

## 授权

## 从授权账户转账

## 查询授权额度