# 事件插件部署 (MongoDB)

***

本文主要介绍事件插件的部署步骤，包括：MongoDB、波场事件订阅插件、波场事件查询服务的部署命令，以及对波场事件查询服务接口的详细介绍。

## 建议配置

- CPU/内存：16核/32G
- 磁盘：500G
- 系统：CentOS 64

## 插件逻辑

- 波场事件订阅插件的功能是从节点获取事件信息并存储到 MongoDB。
- MongoDB 的功能是保存事件信息。
- 波场事件查询服务的功能是提供封装的 HTTP 接口以从 MongoDB 获取事件信息。

## 部署波场事件订阅插件

```shell
# 部署
git clone https://github.com/tronprotocol/event-plugin.git
cd eventplugin
./gradlew build
```

- **配置节点配置文件**

    在节点配置文件的末尾追加以下内容。以下是一个示例，请参阅 README.md。

    ```javascript
    event.subscribe = {
        path = "/deploy/fullnode/event-plugin/build/plugins/plugin-mongodb-1.0.0.zip" // 插件的绝对路径
        server = "127.0.0.1:27017" // 接收事件触发器的目标服务器地址
        dbconfig = "eventlog|tron|123456" // 数据库名称|用户名|密码
        topics = [
            {
              triggerName = "block" // 区块触发器，该值不可修改
              enable = true
              topic = "block" // 插件主题，该值可以修改
            },
            {
              triggerName = "transaction"
              enable = true
              topic = "transaction"
            },
            {
              triggerName = "contractevent"
              enable = true
              topic = "contractevent"
            },
            {
              triggerName = "contractlog"
              enable = true
              topic = "contractlog"
            }
        ]
    
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
    }
    ```

    📘 **字段解析**

    - `path`: 为 `plugin-kafka-1.0.0.zip` 或 `plugin-mongodb-1.0.0.zip` 的绝对路径
    - `server`: Kafka 服务器地址或 MongoDB 服务器地址
    - `dbconfig`: 这是 MongoDB 的配置，对于 Kafka 插件，赋值为空字符串
    - `topics`: 每种事件类型映射到一个 Kafka 主题，支持四种事件类型的订阅：区块、交易、合约日志和合约事件
    - `triggerName`: 触发器类型，该值不可修改
    - `enable`: 如果值为 `false`，插件将接收不到任何信息
    - `topic`: 值为接收事件的 Kafka 主题或 MongoDB 集合。确保其已创建且 Kafka 进程正在运行

## 部署 MongoDB

```shell
# 1、下载并安装 MongoDB
cd /home/java-tron
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.4.tgz
tar zxvf mongodb-linux-x86_64-4.0.4.tgz
mv mongodb-linux-x86_64-4.0.4 mongodb

# 2、设置环境变量
export MONGOPATH=/home/java-tron/mongodb/
export PATH=$PATH:$MONGOPATH/bin

# 3、创建 MongoDB 配置文件
mkdir -p /home/java-tron/mongodb/{log,data}
cd /home/java-tron/mongodb/log/ && touch mongodb.log && cd
vim mgdb.conf
```

将创建的数据和日志文件夹写入配置文件（绝对路径）

**配置文件示例：**

```text
dbpath=/home/java-tron/mongodb/data
logpath=/home/java-tron/mongodb/log/mongodb.log
port=27017
logappend=true
fork=true
bind_ip=0.0.0.0
auth=true
wiredTigerCacheSizeGB=2
```

!!! note
    - `bind_ip` 必须配置为 `0.0.0.0`，否则将拒绝远程连接。
    - `wiredTigerCacheSizeGB` 必须配置以防止 `OOM`。

```shell
# 4、启动 MongoDB
mongod --config ./mgdb.conf &

# 5、创建管理员账户：
mongo
use admin
db.createUser({user:"root",pwd:"admin",roles:[{role:"root",db:"admin"}]})

# 6、创建 eventlog 及其拥有者账户
db.auth("root", "admin")
use eventlog
db.createUser({user:"tron",pwd:"123456",roles:[{role:"dbOwner",db:"eventlog"}]})
```

## 部署波场事件查询服务

```shell
# 1、下载
git clone https://github.com/tronprotocol/tron-eventquery.git cd troneventquery

# 2、安装
wget https://mirrors.cnnic.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz --no-check-certificate
tar zxvf apache-maven-3.5.4-bin.tar.gz
export M2_HOME=$HOME/maven/apache-maven-3.5.4
export PATH=$PATH:$M2_HOME/bin
mvn --version
mvn package
```

命令成功执行后，JAR 包将在 `troneventquery/target` 下生成，配置文件将在 `troneventquery/config.conf` 下生成。配置内容为：

```text
mongo.host=IP
mongo.port=27017
mongo.dbname=eventlog
mongo.username=tron
mongo.password=123456
mongo.connectionsPerHost=8
mongo.threadsAllowedToBlockForConnectionMultiplier=4
```

```shell
# 3、启动波场事件查询服务
sh troneventquery/deploy.sh
sh troneventquery/insertIndex.sh
```

!!! note
    默认端口是 `8080`。如果想要更改端口，请修改脚本 `troneventquery/deploy.sh`：

```shell
nohup java -jar -Dserver.port=8081 target/troneventquery-1.0.0-SNAPSHOT.jar 2>&1 &
```

## 在 Java-tron 中加载插件及验证

```shell
# 1、启动全节点
Java -jar FullNode.jar -c config.conf --es
# 注意：在启动整个节点之前启动 MongoDB。
# 全节点安装参考：https://github.com/tronprotocol/java-tron/blob/develop/build.md

# 2、验证插件加载
tail -f logs/tron.log |grep -i eventplugin
# 出现以下单词即表示成功
# o.t.c.l.EventPluginLoader '你的插件路径/plugin-kafka-1.0.0.zip' loaded

# 3、验证数据是否存储在 MongoDB 中
mongo 47.90.245.68:27017
use eventlog
db.auth("tron", "123456")
show collections
db.block.find()
# 数据成功返回则表示可以通过事件订阅从节点获取数据并存储到 MongoDB 中。否则，检查全节点日志以进行故障排除。
```

## 使用事件查询服务

**主要 HTTP 服务**

```text
# 功能：获取交易列表
子路径: $baseUrl/transactions
参数
limit: 每页大小，默认值为 25
sort: 排序字段，默认按 timeStamp 降序排序
start: 起始页，默认值为 1
block: 起始区块编号，默认值为 0
示例: http://127.0.0.1:8080/transactions?limit=1&sort=-timeStamp&start=2&block=0

# 功能：通过哈希获取交易
子路径: $baseUrl/transactions/{hash}
参数
hash: 交易 ID
示例: http://127.0.0.1:8080/transactions/9a4f096700672d7420889cd76570ea47bfe9ef815bb2137b0d4c71b3d23309e9

# 功能：获取转账列表
子路径: $baseUrl/transfers
参数
limit: 每页大小，默认值为 25
sort: 排序字段，默认按 timeStamp 降序排序
start: 起始页，默认值为 1
from: 发件地址，默认值为 ""
to: 收件地址，默认值为 ""
token: 通证名称，默认值为 ""
示例: http://127.0.0.1:8080/transfers?token=trx&limit=1&sort=timeStamp&start=2&block=0&from=TJ7yJNWS8RmvpXcAyXBhvFDfGpV9ZYc3vt&to=TAEcoD8J7P5QjWT32r31gat8L7Sga2qUy8

# 功能：通过 transactionId 获取转账
子路径: $baseUrl/transfers/{hash}
参数
hash: 转账哈希
示例: http://127.0.0.1:8080/transfers/70d655a17e04d6b6b7ee5d53e7f37655974f4e71b0edd6bcb311915a151a4700

# 功能：获取事件列表
子路径: $baseUrl/events
参数
limit: 每页大小，默认值为 25
sort: 排序字段，默认按 timeStamp 降序排序
since: 事件发生的起始时间，timeStamp >= since 的事件将被显示
start: 起始页，默认值为 1
block: 区块编号，区块编号 >= block 的事件将被显示
示例: http://127.0.0.1:8080/events?limit=1&sort=timeStamp&since=0&block=0&start=0

# 功能：通过 transactionId 获取事件
子路径: $baseUrl/events/transaction/{transactionId}
参数
transactionId
示例: http://127.0.0.1:8080/events/transaction/cd402e64cad7e69c086649401f6427f5852239f41f51a100abfc7beaa8aa0f9c

# 功能：通过合约地址获取事件
子路径: $baseUrl/events/{contractAddress}
参数
limit: 每页大小，默认值为 25
sort: 排序字段，默认按 timeStamp 降序排序
since: 事件发生的起始时间，timeStamp >= since 的事件将被显示
```