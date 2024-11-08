# 内置消息队列订阅 (ZeroMQ)
***

波场提供事件订阅服务。开发者不仅可以通过事件插件获取链上事件，还可以通过 Java-tron 内置的 ZeroMQ 消息队列获取。事件插件需要额外部署，用于实现事件存储：开发者可以根据需要选择合适的存储工具，如 MongoDB、Kafka 等，并通过插件完成订阅事件的存储。Java-tron 内置的 ZeroMQ 无需额外部署操作。事件订阅者可以直接连接到发布者的 IP 和端口，设置订阅主题，接收订阅的事件。然而，这种方法不提供事件存储。因此，当开发者想要直接从节点短期订阅事件时，使用内置消息队列将是更合适的选择。

本文将详细介绍如何通过 Java-tron 内置消息队列订阅事件。

## 配置节点

要使用节点的内置 ZeroMQ 进行事件订阅，需要在节点配置文件中将配置项 `useNativeQueue` 设置为 `true`。

```shell
event.subscribe = {
  native = {
    useNativeQueue = true // 如果为 true，使用内置消息队列，否则使用事件插件。
    bindport = 5555 // 绑定端口
    sendqueuelength = 1000 // 发送队列的最大长度
  }

  ......
 
  topics = [
    {
      triggerName = "block" // 区块触发器，该值不可修改
      enable = true
      topic = "block" // 插件主题，该值可以修改
    },
    ......
  ]
}
```

- `native.useNativeQueue`: `true` 表示使用内置消息队列，`false` 表示使用事件插件。
- `native.bindport`: ZeroMQ 发布者绑定端口。在此示例中为 `5555`，所以订阅者应连接的发布者地址是 `tcp://127.0.0.1:5555`。
- `native.sendqueuelength`: 发送队列的长度，即当订阅者接收消息较慢时，发布者发布的最大消息数即 TCP 缓冲区可以容纳的数量。如果超过容量，会丢弃消息。
- `topics`: 订阅的事件类型，包括区块类型、交易类型等。

## 启动节点

事件订阅服务默认关闭，需要通过添加命令行参数 `--es` 启用。启用事件订阅服务的节点启动命令如下：

```shell
$ java -jar FullNode.jar --es
```

## 准备事件订阅脚本

本文以 Nodejs 为例说明如何订阅事件。

首先，安装 `zeromq` 库：

```shell
$ npm install zeromq@5
```

然后，编写订阅者代码：

```javascript
// subscriber.js
var zmq = require("zeromq"),
var sock = zmq.socket("sub");

sock.connect("tcp://127.0.0.1:5555");
sock.subscribe("block");
console.log("Subscriber connected to port 5555");

sock.on("message", function(topic, message) {
  console.log(
    "received a message related to:",
    Buffer.from(topic).toString(),
    ", containing message:",
    Buffer.from(message).toString()
  );
});
```

此示例将订阅者连接到节点事件发布者，并订阅区块事件。

## 启动订阅者

Nodejs 的启动命令如下：

```shell
$ node subscriber.js

> Subscriber connected to port 5555
```

当节点有新区块时，订阅者将收到区块事件，输出信息如下：

```shell
received a message related to: blockTrigger, containing message: {"timeStamp":1678343709000,"triggerName":"blockTrigger","blockNumber":1361,"blockHash":"00000000000005519b3995cd638753a862c812d1bda11de14bbfaa5ad3383280","transactionSize":0,"latestSolidifiedBlockNumber":1361,"transactionList":[]}
received a message related to: blockTrigger, containing message: {"timeStamp":1678343712000,"triggerName":"blockTrigger","blockNumber":1362,"blockHash":"0000000000000552d53d1bdd9929e4533a983f14df8931ee9b3bf6d6c74a47b0","transactionSize":0,"latestSolidifiedBlockNumber":1362,"transactionList":[]}
```