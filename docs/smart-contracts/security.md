# 安全性
***
智能合约灵活性极强，能够持有大量通证，并基于先前部署的智能合约代码运行不可变的逻辑。虽然这创造了一个充满活力且富有创意的去信任化、互联互通的智能合约生态系统，但也成为了吸引攻击者的完美生态系统，这些攻击者试图通过利用智能合约中的漏洞和波场网络中的意外行为来获利。智能合约代码通常无法更改以修补安全漏洞，从智能合约中被盗的资产无法追回，且被盗资产极难追踪。

在将任何代码发布到主网之前，采取足够的预防措施以保护智能合约中的有价值资产非常重要。在本文中，我们将讨论一些具体的攻击和最佳实践，以确保合约能够正确且安全地运行。

## 智能合约开发过程

安全性始于恰当的设计和开发过程。在智能合约开发过程中需要牢记许多事项，但至少应确保以下几点：

- 所有代码存储在版本控制系统中，如 git
- 所有代码修改通过拉取请求 (Pull Requests) 进行
- 所有拉取请求至少有一名审阅者
- 使用开发工具（例如 tronbox）通过单个命令编译、部署和运行一套针对代码的测试
- 在合并每个拉取请求之前，已使用诸如 Mythril 和 Slither 等基本代码分析工具检查代码，将输出结果进行比较
- Solidity 不发出任何编译器警告
- 代码有良好的文档记录

## 攻击和漏洞

以下是一些常见的漏洞：

### 重入攻击

重入攻击是开发智能合约时需要考虑的最重要的安全问题之一。
虽然 TVM 不能同时运行多个合约，但一个合约调用不同合约会暂停调用合约的执行和内存状态，直到调用返回，此时执行才能正常进行。
这种暂停和重新开始可能会导致一种被称为“重入”的漏洞。

以下是易受重入攻击的合约的简单版本：

```solidity
// 此合约存在故意设计的漏洞，请勿复制
contract Victim {
    mapping (address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success);
        balances[msg.sender] = 0;
    }
}
```

为了允许用户提取其之前在合约中存储的 TRX，`withdraw` 函数将按以下顺序执行：

1. 读取用户的余额
2. 向用户发送该余额的 TRX
3. 将其余额重置为 0，以防止再次提取

如果从常规外部账户调用（例如您自己的 Tronlink 账户），此函数按预期运行：`msg.sender.call.value()` 仅将 TRX 发送到您的账户。
然而，智能合约也可以进行调用。如果是自定义的恶意合约调用 `withdraw()`，`msg.sender.call.value()` 不仅会发送 TRX，还会隐式调用合约以开始执行代码。
假设存在以下恶意合约：

```solidity
contract Attacker {
    uint count;
    function beginAttack() external payable {
        count = 5;
        Victim(VICTIM_ADDRESS).deposit.value(1 trx)();
        Victim(VICTIM_ADDRESS).withdraw();
    }

    function() external payable {
        if (count > 0) {
            count -= 1;
            Victim(VICTIM_ADDRESS).withdraw();
        }
    }
}
```

调用 `Attacker.beginAttack()` 将启动一个过程，如下所示：

```
0. 攻击者的外部账户调用 `Attacker.beginAttack()`，并携带 1 TRX。
0. `Attacker.beginAttack()` 向受害者合约存入 1 TRX：`Victim.deposit.value(1 trx)()`。

1. 攻击者调用受害者的 `withdraw` 函数：`Victim.withdraw()`
1. 受害者读取调用者的余额 1 TRX：`balances[msg.sender]`
1. 受害者向攻击者发送 TRX，执行攻击者合约的默认函数调用
    2. 在攻击者的默认函数中执行 `Victim.withdraw()`
    2. 受害者读取余额：`balances[msg.sender]`
    2. 受害者向攻击者发送 TRX，执行攻击者合约的默认函数调用
      3. 在攻击者的默认函数中执行 `Victim.withdraw()`
      3. 受害者读取余额：`balances[msg.sender]`
      3. 受害者向攻击者发送 TRX，执行攻击者合约的默认函数调用
        4. 为了不超过合约允许的最大执行时间，攻击者在执行多次后停止执行 `withdraw` 并直接返回。
      3. `balances[msg.sender] = 0;`
    2. `balances[msg.sender] = 0;`
  1. `balances[msg.sender] = 0;`
```

调用 `Attacker.beginAttack` 带有 1 TRX 将对受害者进行重入攻击，提取的 TRX 超过了提供的数量。即，攻击者从其他用户的余额中窃取 TRX。

### 如何应对重入攻击

通过简单地调整存储更新和外部调用的顺序，可以防止允许攻击的重入条件。在以下示例中，`withdraw` 函数首先将存储的余额信息设置为 0，然后再转移 TRX，以避免恶意代码重入攻击。

```solidity
contract NoLongerAVictim {
    function withdraw() external {
        uint256 amount = balances[msg.sender];
        balances[msg.sender] = 0;
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success);
    }
}
```

每当您向不可信地址发送 TRX 或与不明合约交互时（例如调用用户提供的通证地址的 `transfer()`），您将面临重入的可能性。通过设计不发送 TRX 也不调用不可信合约的合约，可以防止重入的可能性！

### 更多攻击类型

除了上述由于智能合约编码引起的重入攻击，还有许多其他类型的攻击，例如：

- TRX 发送拒绝
- 整数溢出/下溢