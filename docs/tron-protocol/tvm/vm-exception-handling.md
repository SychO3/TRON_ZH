# 虚拟机异常处理
***
## 异常类型
合约执行过程中可能出现四种例外情况：

1. [Asset-style 异常](#asset-style)
2. [Require-style 异常](#require-style)
3. [Validation-style 异常](#validation-style)
4. [VMillegal-style 异常](#vmillegal-style)

### Asset-style 异常
下面的代码会产生一个 Asset 类型的异常，抛出一个无效操作码错误，并消耗所有能量（包括迄今为止已消耗和未消耗的能量）：

1. 如果访问的数组索引过大或为负数（例如 x[i]，其中 i >= x.length 或 i < 0）。
2. 如果访问固定长度的字节N，索引过大或为负数。
3. 在除法或模数运算中使用零作为除数（例如 5 / 0 或 23 % 0）。
4. 移位负数。
5. 将过大或过小的负值转换为枚举类型。
6. 调用未初始化的内部函数类型变量。
7. 调用断言（表达式）的参数，且最终结果为假。
8. 合约执行过程中超时。
9. 发生 JVMStackOverFlowException 异常。
10. 发生 OutofMem 异常，即内存超出 3M。
11. 在合约操作过程中发生溢出，如加法。

### Require-style 异常
以下情况会产生 require 类型的异常，引发还原错误，并且只消耗已消耗的能量，不包括未消耗的能量。

1. 调用 `throw`
2. 如果调用 `require` 参数（表达式），最终结果为 `false`。
3. 如果通过信息调用函数，但函数没有正确结束（例如，函数耗尽能量或自身抛出异常）。如果在调用函数时没有指定能量值，所有能量都会被传入并消耗。只有设置能量值才能看到差异。该函数不包括调用、发送、委托调用或调用代码等低级操作。低级操作不会抛出异常，但会返回 false 表示失败。
4. 如果使用 new 关键字创建合约，但合约创建不正确（因为在创建合约时无法指定能量，所有能量都会传入并消耗）。
5. 如果合约通过没有可支付修饰符的公共函数接收 TRX（包括构造函数、回退函数和通用公共函数）。
6. `transfer()` 失败
7. 调用 `revert()`
8. 达到最大函数堆栈深度 64。

!!!note
    断言式和要求式情况都会导致 TVM 回退。回退的原因是，由于没有达到预期的效果，因此无法继续安全地执行。由于我们希望保持事务的原子性，所以最安全的做法是回滚所有更改。不过，我们会进行扣减。

### Validation-style 异常
以下情况将导致 Validation 类型的异常。事务不会被链式处理，也不会消耗任何能量。

1. 当前版本不支持虚拟机。
2. 创建合同时，合同名称超过 32 字节。
3. 创建合约时，消耗的调用者资源比例不在 [0, 100] 之间
4. 创建合约时，新生成的合约地址发生哈希冲突，即合约地址已生成。
5. 余额不足时，调用值不是 0。
6. 费用上限不在合法范围内。
7. 向不支持常量的节点发送常量请求。
8. 数据库中不存在触发的合同。

### VMillegal-style 异常
在以下情况下会出现 VMillegal 类型的异常。这类异常事务不会被链式处理，但发送事务的节点会在网络层受到一段时间的处罚。

1. 创建合约时，`OwnerAddress` 和 `OriginAddress` 不相等。
2. 广播常数请求。



## 异常处理流程
条目是 go() 异常，将在 go() 中捕获和处理，不会传递到 go() 以外。

```javascript
public void go() {

    try {
      vm.play(program);

      result = program.getResult();

      // If there is Exception or Revert
      // Important：
      // Exception is thrown in the program settings. 
      // Revert is a Virtual Machine compiler written into bytecode in advance, arriving by jump. 
      if (result.getException() != null || result.isRevert()) {

        if (result.getException() != null) {
          // If Exception，will consume all Energy
          program.spendAllEnergy();
          // Set runtimeError to indicate the field of the error content
          runtimeError = result.getException().getMessage();
          // Throw an exception
          throw result.getException();
        } else {
          // If it is Revert and there is no Exception, just set the runtimeError field to indicate the error content. 
          runtimeError = "REVERT opcode executed";
        }

        // As long as Exception or Revert occurs, it will not commit. All state changes in the virtual machine execution process will not fall. 

      } else {
        // Without Exception and Revert, commit, all state changes during virtual machine execution will fall. 
        deposit.commit();
      }
    }
    catch (JVMStackOverFlowException e) {
        // TVM or JVM, JVMStackOverFlowException, flags exception. 
        // The JVMStackOverFLowException will only be caught in go(), which uses JVMStackOverFlowException that occurs in the contract called by call. Or the JVMStackOverFlowException that is called repeatedly. It won't catch up in play(), and will only be caught here. 
        result.setException(e);
        runtimeError = result.getException().getMessage();
    }
    // Catch all the content that can be thrown
    catch (Throwable e) {
      // Mark if the exception is unknown. 
      if (Objects.isNull(result.getException())) {
        result.setException(new RuntimeException("Unknown Throwable"));
      }
      // Ensures the runtimeError has a value
      if (StringUtils.isEmpty(runtimeError)) {
        runtimeError = result.getException().getMessage();
      }

    }
  }
  // After the go() function, result.getException() will not be used, and runtimeError will be filled in the transactionInfo.
```

play() 函数是虚拟机实际执行的地方。有三个地方会调用 play()，分别是 go()（如上所述）、callToAddress()（调用 CALL 指令，在合约中调用）和 createContract()（CREATE 指令，在合约中创建合约时调用）。后两者将不处理捕获异常，在游戏中抛出的异常将继续在后两者中抛出。

```javascript
// Play will catch all RuntimeException, and throw a JVMStackOverFlowException (including StackOverflowError)
  public void play(Program program) {
    try {
      // An op virtual execution virtual machine
      while (!program.isStopped()) {
        // Step will first catch the RuntimeException, deduct the Energy, and then throw. 
        // At this time, it will stop the step and catch the RuntimeException first, then deduct the Energy ring. 
        this.step(program);
      }
    }
    catch (JVMStackOverFlowException e) {
      // Throw a JVMStackOverFlowException exception
      throw new JVMStackOverFlowException();
    } catch (RuntimeException e) {
      if (StringUtils.isEmpty(e.getMessage())) {
        // Put the RuntimeException thrown by the step() function into program.result.exception instead of throwing it out. 
        program.setRuntimeFailure(new RuntimeException("Unknown Exception"));
      } else {
        program.setRuntimeFailure(e);
      }
    } catch (StackOverflowError soe) {
      // Throw a JVMStackOverFLowException exception
      throw new JVMStackOverFlowException();
    } finally {
    }
  }
```

step() 函数

```javascript
// Step first catches the RuntimeException, deducts the Energy, and then throws
// Note, the exceptions thrown here are all RuntimeException
public void step(Program program) {
  try {
    // If the op is illegal, an IllegalOperationException is thrown. In fact, the Invalid is written in advance in the bytecode, and the assert-style operation jumps to invalid. 
    OpCode op = OpCode.code(program.getCurrentOp());
    if (op == null) {
      throw Program.Exception.invalidOpCode(program.getCurrentOp());
    }
    switch (op) {
      // 1. Calculate the energy required for the op

      // 2. If deduction is not enough, throw an OutOfEnergyException
      program.spendEnergy(energyCost, op.name());
 
      // 3. Detect CPU time, timeout, then throw OutOfResourceException
      program.checkCPUTimeLimit(op.name());

      // 4. Actual execution of OP
      // The focus here is on the CREATE instruction and the CALL instruction. 
      // The steps inside are all similar: 
      // 4.1 When the depth is up，it will push 0 to stack，then return
      // 4.2 If there is value, the balance is insufficient, then push 0 to stack, and return
      // 4.3 If there is value, transfer failes, throwing an exception of type RuntimeException. 
      // 4.4 (Requires execution) to execute the virtual machine
      // 4.5 The result of executing the virtual machine, there is an exception. No exception will be thrown, only all Energy will be deducted and push 0 to stack. 
      // 4.6 The result of executing the virtual machine, if there is a revert, it will return Energy, and push 0 to stack. 
      // 4.7 Successful execution will return Energy and push 1 to stack.

      // Note：
      // Revert operation, and after exception, how to deal with, is determined by the virtual machine bytecode. Some will revert and some will be invalid. 
      // callToPrecompile fails, it will directly throw an exception of RuntimeException type
   }
 } catch (RuntimeException e) {
   // step will first catch the RuntimeException, deduct the Energy, and then throw
   program.spendAllEnergy();
   // Stop loop
   program.stop();
    // Throw an exception
    throw e;
  } finally {
  }
}
```





