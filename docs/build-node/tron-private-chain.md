# 波场私有链
构建波场私有链
***

要构建私有链，至少需要部署一个由超级代表（SR）运行的全节点来产生区块，以及任意数量的全节点来同步区块和广播交易。
在此示例中，仅设置一个超级代表节点和一个全节点。

## **准备工作**
- Oracle JDK 1.8
- 创建至少两个波场网络地址并保存这些地址和私钥。可以使用 tronweb、wallet-cli 或 Tronlink 创建地址。

## **部署指南**
在私有链上构建节点的过程与主网上相同。不同之处在于节点配置文件的内容。构建私有链最重要的一步是修改配置文件中的配置项，使节点能够形成一个用于节点发现、区块同步和广播交易的私有网络。

1. 创建部署目录  
    创建部署目录，建议将两个全节点放在不同目录中。

    ```bash
    $ mkdir SR $ mkdir FullNode
    ```

2. 获取 FullNode.jar，然后分别放入 SR 和 FullNode 目录。

    ```bash
    $ cp FullNode.jar ./SR
    $ cp FullNode.jar ./FullNode
    ```

3. 获取节点的配置文件 private_net_config.conf，并将其分别放入 SR 和 FullNode 目录中，并分别修改文件名为 supernode.conf 和 fullnode.conf。

    ```bash
    $ cp private_net_config.conf ./SR/supernode.conf
    $ cp private_net_config.conf ./FullNode/fullnode.conf
    ```

4. 修改每个节点的配置文件

    | 配置项 | 超级代表全节点 | 全节点 | 描述 |
    | ------ | ------------- | ----- | ---- |
    | localwitness | 见证地址的私钥 | 请勿填写数据 | 生成区块需要使用私钥进行签名 |
    | genesis.block.witnesses | 见证地址 | 与超级代表节点相同 | 创世区块相关配置 |
    | genesis.block.Assets | 为特定账户预设 TRX。将预设地址添加到末尾，并按需指定其 TRX 余额 | 与超级代表节点相同 | 创世区块相关配置 |
    | p2p.version | 除 11111 外的任意正整数 | 与超级代表节点相同 | 只有相同 p2p 版本的节点才能成功握手 |
    | seed.node | 请勿填写数据 | 将配置文件中的 seed.node ip.list 修改为超级代表节点的 IP 地址和端口（listen.port） | 使全节点能够与超级代表节点建立连接并同步数据 |
    | needSyncCheck | false | true | 设置第一个超级代表的 needSyncCheck 为 false，其他超级代表为 true |
    | node.discovery.enable | true | true | 如果为 false，当前节点将不会被其他节点发现 |
    | block.proposalExpireTime | 600000 | 与超级代表节点相同 | 默认提案有效时间为3天：259200000 ms。如果想快速通过提案，可以将此项设置为较小值如10分钟，即 600000 ms |
    | block.maintenanceTimeInterval | 300000 | 与超级代表节点相同 | 默认维护时间间隔为6小时：21600000 ms。如果想快速通过提案，可以将此项设置为较小值如五分钟，即 300000 ms |
    | committee.allowSameTokenName | 1 | 1 | 允许相同通证名称 |
    | committee.allowTvmTransferTrc10 | 1 | 1 | 允许 TVM 转账 TRC10 |

5. 修改配置文件中的端口，并为超级代表和全节点配置不同的端口号。如果超级代表和全节点运行在同一台机器上，此步骤必不可少，否则可以跳过。

     - listen.port：p2p 监听端口
     - http port：HTTP 监听端口
     - rpc port：RPC 监听端口

6. 启动节点

    - 生产区块的全节点：

    ```bash
    $ java -Xmx6g -XX:+HeapDumpOnOutOfMemoryError -jar FullNode.jar --witness -c supernode.conf
    ```

    - 全节点：

    ```bash
    $ java -Xmx6g -XX:+HeapDumpOnOutOfMemoryError -jar FullNode.jar -c fullnode.conf
    ```

7. 修改私有链的动态参数

    为了与主网环境一致，需要将私有链的动态参数修改为与主网一致。动态参数的修改可以通过提案完成。超级代表账户可以使用 tronweb、wallet-cli 或全节点 HTTP API `wallet/proposalcreate` 创建提案，使用 `wallet/proposalapprove` 批准提案。

    以下是根据主网先后通过的提案整理出的动态参数及值。超级代表可以直接使用以下命令创建提案，以完成私有链所有动态参数的修改。由于一些参数之间的依赖关系，根据主网当前的参数值，将私有链所有参数的修改分为两个提案。第一步，超级代表根据以下代码创建并投票第一个提案：

    ```javascript
    var TronWeb = require('tronweb');
    var tronWeb = new TronWeb({
        fullHost: 'http://localhost:16887',
        privateKey: 'c741f5c0224020d7ccaf4617a33cc099ac13240f150cf35f496db5bfc7d220dc'
    })
    
    // 第一个提案："key":30 和 "key":70 必须首先修改
    var parametersForProposal1 = [
        {"key":9,"value":1},{"key":10,"value":1},{"key":11,"value":420},{"key":19,"value":90000000000},
        {"key":15,"value":1},{"key":18,"value":1},{"key":16,"value":1},{"key":20,"value":1},
        {"key":26,"value":1},{"key":30,"value":1},{"key":5,"value":16000000},{"key":31,"value":160000000},
        {"key":32,"value":1},{"key":39,"value":1},{"key":41,"value":1},{"key":3,"value":1000},
        {"key":47,"value":10000000000},{"key":49,"value":1},{"key":13,"value":80},{"key":7,"value":1000000},
        {"key":61,"value":600},{"key":63,"value":1},{"key":65,"value":1},{"key":66,"value":1},
        {"key":67,"value":1},{"key":68,"value":1000000},{"key":69,"value":1},{"key":70,"value":14},
        {"key":71,"value":1},{"key":76,"value":1}
    ];
    var parametersForProposal2 = [
        {"key":47,"value":15000000000},{"key":59,"value":1},{"key":72,"value":1},{"key":73,"value":3000000000},
        {"key":74,"value":2000},{"key":75,"value":12000},{"key":77,"value":1},{"key":78,"value":864000}
    ];
    
    async function modifyChainParameters(parameters, proposalID) {
        parameters.sort((a, b) => {
            return a.key.toString() > b.key.toString() ? 1 : a.key.toString() === b.key.toString() ? 0 : -1;
        })
        var unsignedProposal1Txn = await tronWeb.transactionBuilder.createProposal(parameters, "41D0B69631440F0A494BB51F7EEE68FF5C593C00F0")
        var signedProposal1Txn = await tronWeb.trx.sign(unsignedProposal1Txn);
        var receipt1 = await tronWeb.trx.sendRawTransaction(signedProposal1Txn);
    
        setTimeout(async function() {
            console.log(receipt1)
            console.log("Vote proposal 1 !")
            var unsignedVoteP1Txn = await tronWeb.transactionBuilder.voteProposal(proposalID, true, tronWeb.defaultAddress.hex)
            var signedVoteP1Txn = await tronWeb.trx.sign(unsignedVoteP1Txn);
            var rtn1 = await tronWeb.trx.sendRawTransaction(signedVoteP1Txn);
        }, 1000)
    }
    
    modifyChainParameters(parametersForProposal1, 1)
    ```

通过上述代码创建提案后，可以通过 `http://127.0.0.1:xxxx/wallet/listproposals` 接口查询提案的生效时间："expiration_time"。该时间戳单位为毫秒。在生效时间过后，如果接口返回值中的 "state" 为 "APPROVED"，则表示提案已通过，可以继续进行下一步并创建第二个提案。示例代码如下：

```javascript
modifyChainParameters(parametersForProposal2, 2)
```

提案生效后，私有链的动态参数将与主网一致。可以通过 `/wallet/getchainparameters` API 查询链参数。