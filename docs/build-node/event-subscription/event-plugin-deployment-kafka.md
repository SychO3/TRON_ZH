# 事件插件部署 (Kafka)
***


本页面提供部署 Kafka 事件订阅插件的说明。内容包括：插件编译、Kafka 部署和事件订阅示例。

## 推荐配置

- CPU/内存：16核/32G
- 磁盘：500G+
- 系统：Ubuntu / CentOS 64

## 1. 编译 Kafka 事件插件

```java
git clone https://github.com/tronprotocol/event-plugin.git
cd event-plugin
./gradlew build
```

编译后可以在 `event-plugin/build/plugins/` 目录中找到 `plugin-kafka-1.0.0.zip`。

## 2. 部署 Kafka

### **安装 Kafka**

```java
cd /usr/local
wget https://downloads.apache.org/kafka/2.8.0/kafka_2.13-2.8.0.tgz
tar -xzf kafka_2.13-2.8.0.tgz
```

### **启动 Kafka**

```java
cd /usr/local/kafka_2.13-2.8.0
# 启动 ZooKeeper 服务
bin/zookeeper-server-start.sh config/zookeeper.properties &
# 启动 Kafka broker 服务
bin/kafka-server-start.sh config/server.properties &
```

## 3. 事件订阅

### 3.1 事件订阅配置

将 `event.plugin` 项目添加到您的全节点配置文件以支持使用 Kafka 的事件订阅。

### 3.1.1 插件配置

```shell
event.subscribe = {
  native = {
    useNativeQueue = false // 如果为 true，使用本地消息队列，否则使用事件插件。
    bindport = 5555 // 绑定端口
    sendqueuelength = 1000 // 发送队列的最大长度
  }

  path = "" // 插件的绝对路径
  server = "" // 接收事件触发器的目标服务器地址
  dbconfig = "" // 数据库名称|用户名|密码
  contractParse = true,
  …………
  …………
}
```

- `native` 包含波场内置消息队列的配置。请始终将 `useNativeQueue` 保持为 `false` 以使用 Kafka 订阅事件。
- `path` 是 `plugin-kafka-1.0.0.zip` 的本地路径。请确保路径正确。
- `server` 是以 IP:port 格式的 Kafka 服务器地址，默认端口是 `9092`。请使用正确的端口号并确保 Kafka 服务可访问。
- `dbconfig` 是用于 MongoDB 的，可以忽略。

### 3.1.2 事件订阅配置项

波场事件订阅支持七种类型的事件。分别是：

- `block`
- `transaction`
- `contractevent`
- `contractlog`
- `solidity`
- `solidityevent`
- `soliditylog`

订阅过多的主题会导致性能下降。请根据需要订阅 1-2 种类型的事件。

以区块事件订阅为例：

```shell
topics = [
    {
      triggerName = "block" // 区块触发器，该值不可修改
      enable = true
      topic = "block" // 插件主题，该值可以修改
    }
]
```

- `triggerName`: 内置字段。不可更改。
- `enable`: 设置为 `true` 以订阅 `block` 事件。
- `topic`: 此字段对应于 Kafka 中预创建的主题。主题名称可以是默认的或自定义的。

`filter` 项目用于过滤订阅的事件。为了精确订阅，根据需要可以自定义区块范围（`fromblock ~ toblock`）、合约地址（`contractAddress`）和特定合约主题（`contractTopic`）。

```shell
filter = {
    fromblock = "" // 值可以是 ""、"earliest" 或指定的区块编号，作为查询范围的开始
    toblock = "" // 值可以是 ""、"latest" 或指定的区块编号，作为查询范围的结束
    contractAddress = [
      "" // 想要订阅的合约地址，如果设置为 ""，将接收任何合约地址的合约日志/事件。
    ]

    contractTopic = [
      "" // 想要订阅的合约主题，如果设置为 ""，将接收任何合约主题的合约日志/事件。
    ]
}
```

### 3.2 在 Kafka 中创建订阅主题

Kafka 订阅主题的名称应与 3.1.2 中的配置项一致。例如，要订阅事件 `block`，请在 `block trigger` 中将 `topic` 设置为 `block`，并在 Kafka 中创建主题 `block` 以接收 `block` 事件。

Linux:

```java
bin/kafka-topics.sh --create --topic block --bootstrap-server localhost:9092
```

### 3.3 启动事件订阅节点

完成上述配置后，请使用 `--es` 启动您的全节点以开启事件订阅。

```java
java -jar FullNode.jar -c config.conf --es
```

检查 tron.log 查看 Kafka 事件插件是否已成功加载。

```java
grep -i eventplugin logs/tron.log
```

显示以下提示时即表示加载成功：

```shell
[o.t.c.l.EventPluginLoader] '你的插件路径/plugin-kafka-1.0.0.zip' loaded
```

### 3.4 事件订阅查询

执行 `kafka-console-consumer.sh` 脚本以获取 Kafka 中主题 `block` 的消息。

Linux:

```java
bin/kafka-console-consumer.sh --topic block --from-beginning --bootstrap-server localhost:9092
```

出现以下内容时表示成功订阅事件：

```shell
{
	"timeStamp": 1539973125000,
	"triggerName": "blockTrigger",
	"blockNumber": 3341315,
	"blockHash": "000000000032fc03440362c3d42eb05e79e8a1aef77fe31c7879d23a750f2a31",
	"transactionSize": 16,
	"latestSolidifiedBlockNumber": 3341297,
	"transactionList": ["8757f846e541b51b5692a2370327f4b8031125f4557f8ad4b1037d4452616d39", "f6adab7814b34e5e756170f93a31a0c3393c5d99eff11e30271916375adc7467", ..., "89bcbcd063a48ef4a5678a033acf5edbb6b17419a3c91eb0479a3c8598774b43"]
}
```