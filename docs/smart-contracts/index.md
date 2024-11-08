# 介绍

***

## 智能合约

“智能合约”是运行在波场网络上的应用程序。
它是位于波场网络特定账户地址的一组代码（其功能）和数据（其状态）。
智能合约是一种波场账户。这意味着它们有余额，并且可以通过网络发送交易。
然而，它们不是由用户控制的，而是部署到网络并按编程运行。
用户账户可以通过提交执行智能合约上定义函数的交易来与智能合约交互，但与它们的交互是不可逆的。

也许智能合约最好的比喻是自动售货机。
输入正确，某个输出就有保证。
这种逻辑被编程到自动售货机中：金钱+零食选择=零食发放，然后用户可以从自动售货机获取零食。

智能合约就像自动售货机一样，具有编程的逻辑。以下是一个简单的智能合约示例——自动售货机：

```solidity
pragma solidity 0.8.7;

contract VendingMachine {

    // 声明合约的状态变量
    address public owner;
    mapping (address => uint) public cupcakeBalances;

    // 当“VendingMachine”合约被部署时：
    // 1. 设置部署地址为合约所有者
    // 2. 设置部署的智能合约的纸杯蛋糕余额为100
    constructor() {
        owner = msg.sender;
        cupcakeBalances[address(this)] = 100;
    }

    // 允许所有者增加智能合约的纸杯蛋糕余额
    function refill(uint amount) public {
        require(msg.sender == owner, "只有所有者可以补货。");
        cupcakeBalances[address(this)] += amount;
    }

    // 允许任何人购买纸杯蛋糕
    function purchase(uint amount) public payable {
        require(msg.value >= amount * 1 trx , "你必须至少支付1TRX每个纸杯蛋糕");
        require(cupcakeBalances[address(this)] >= amount, "库存中没有足够的纸杯蛋糕来完成此购买");
        cupcakeBalances[address(this)] -= amount;
        cupcakeBalances[msg.sender] += amount;
    }
}
```

就像自动售货机消除了售货员的需要，智能合约可以在许多行业中取代中介。


## 智能合约的属性

波场网络上的智能合约具有以下属性：

- **无需许可**：任何人都可以编写智能合约并将其部署到波场网络。您只需要学习如何使用智能合约语言进行编码，并拥有足够的 TRX 来部署合约。部署智能合约在技术上是一个交易，因此您需要支付资源费用，就像您需要为简单的 TRX 转账支付费用一样。然而，合约部署所消耗的资源要高得多。

    波场提供了一个对开发者友好的智能合约编写语言：Solidity。然而，在部署之前它们必须被编译，以便波场虚拟机能够解释并存储合约。

- **可组合性**：波场网络上的智能合约是公开的，可以被视为开放的 API。这意味着您可以在自己的智能合约中调用其他智能合约，从而极大地扩展可能性。合约甚至可以部署其他合约。您无需编写自己的智能合约即可成为一个去中心化应用开发者，您只需了解如何与它们交互。例如，您可以使用 SunSwap（一种去中心化交易所）的现有智能合约来处理您应用中的所有通证交换逻辑——无需从头开始。


## 智能合约的局限性

波场网络的智能合约具有以下局限性：

 - **无法与外部系统通信**：智能合约无法直接与外部系统通信，因此它们本身无法获取“现实世界”事件的信息，这一瓶颈限制了智能合约的应用场景，但这是有意为之。依赖外部信息可能会危及共识，而共识对安全性和去中心化至关重要。不过，可以使用预言机来解决此问题。

 - **智能合约的最大执行时间**：为了确保网络的高吞吐量和稳定运行，波场将 TVM 的最大执行时间设置为80毫秒，以确保波场网络每3秒可以生成一个新块，因此智能合约允许的最大执行时间为80毫秒。TVM 的最大执行时间是波场网络的第13个动态参数，超级代表委员会可以通过发起提案来修改此参数。

    对于复杂的智能合约，执行可能会超时并触发 OUT_OF_TIME 错误，调用者将被扣除全部 fee_limit 费用。因此，为了避免智能合约执行超时，尝试将大型合约拆分为较小的合约并根据需要相互引用，并注意常见的陷阱和递归调用以避免无限循环。