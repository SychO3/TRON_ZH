# 部署节点
***
支持的操作系统：Linux、MacOS

工具和依赖项：Oracle JDK 1.8、git

## 推荐配置

- CPU：16核
- 内存：32G
- SSD：2.5T+
- 带宽：100M

如果您是作为超级代表来构建用于区块生产的全节点，推荐配置是：CPU：32核，内存：64G


## 部署指南  
无论节点类型如何，部署过程都相同，请参考以下步骤：

### 1. 获取 FullNode.jar  
您可以通过编译源代码或直接下载[发布的 jar](https://github.com/tronprotocol/java-tron/releases) 来获取 FullNode.jar。
   
**编译源代码**  

1. 获取 java-tron 源代码  
    ```shell
       $ git clone https://github.com/tronprotocol/java-tron.git
       $ git checkout -t origin/master
    ```
   
2. 编译  
    ```shell
       $ cd java-tron
       $ ./gradlew clean build -x test
    ```

如果构建成功，您将在 `./java-tron/build/libs/` 文件夹下找到 FullNode.jar。

### 2. 启动节点  
获取主网配置文件：[main_net_config.conf](https://github.com/tronprotocol/tron-deployment/blob/master/main_net_config.conf)，
其他网络配置文件可以在[这里](https://github.com/tronprotocol/tron-deployment)找到。

- **启动主网全节点** 
    全节点拥有完整的历史数据，是进入波场网络的入口，提供 HTTP API 和 Grpc API 以供外部查询。您可以通过全节点与波场网络进行交互：转移资产、部署合约、与合约交互等。主网全节点的启动命令如下，并通过 -c 参数指定全节点的配置文件：


    ```shell
    $ java -Xmx24g -XX:+UseConcMarkSweepGC -jar FullNode.jar -c main_net_config.conf
    ```

    + `-XX:+UseConcMarkSweepGC`：指定并行垃圾回收。需放在 `-jar` 参数之前，而不是最后。
    + `-Xmx`：JVM 堆的最大值，可设置为物理内存的 80%。

- **启动用于主网出块的全节点**  
    在启动命令中添加 `--witness` 参数，全节点将作为出块节点运行。除了支持全节点的所有功能外，出块全节点还支持区块生产和交易打包。请确保您有一个超级代表账户并获得其他人的投票。如果投票排名进入前 27 名，则需要启动一个出块的全节点以参与区块生产。

    将超级代表地址的私钥填写到 `main_net_config.conf` 中的 `localwitness` 列表中，以下是一个示例。但如果您不想以明文方式指定私钥，可以使用密钥库加密码的方法，请参阅“其他”章节。

    ```plaintext
    localwitness = [
       650950B193DDDDB35B6E48912DD28F7AB0E7140C1BFDEFD493348F02295BD812
    ]
    ```

    然后运行以下命令启动节点：

    ```shell
    $ java -Xmx24g -XX:+UseConcMarkSweepGC -jar FullNode.jar --witness -c main_net_config.conf
    ```

注意：对于主网和Nile测试网，由于新节点启动后需要同步的数据量较大，数据同步将耗费较长时间。您可以使用数据快照来加速节点同步。首先下载最新的数据快照，并将其解压到 Tron 项目的 output-directory 目录中，然后启动节点，使节点在数据快照的基础上进行同步。

对于一个运行中的全节点，您可以使用命令 `kill -15` 进程ID 来关闭它。

## 其他事项  
### 如何使用密钥库加密码来指定见证账户的私钥

1. 由于运行节点时需要交互，您不应使用 `nohup` 命令。建议使用 `session` 保持工具，比如 `screen`、`tmux` 等。  
2. 注释掉 `main_net_config.conf` 中的 `localwitness` 项，并取消 `localwitnesskeystore` 项的注释。 填写密钥库文件的路径。

    !!! note
        密钥库文件需要放置在执行启动命令的当前目录或其子目录中。如果当前目录是 "A"，密钥库文件的目录是 "A/B/localwitnesskeystore.json"，则需要配置为：
    
        ```plaintext
        localwitnesskeystore = [
            "B/localwitnesskeystore.json"
        ]
        ```
    
    !!! note
        对于密钥库加密码的生成，您可以使用 wallet-cli 的 register wallet 命令。  

3. 启动出块的全节点

    ```shell
    $ java -Xmx24g -XX:+UseConcMarkSweepGC -jar FullNode.jar --witness -c main_net_config.conf
    ```

4. 正确输入密码以完成节点启动。  


### 使用 tcmalloc 优化内存分配

可以使用 `tcmalloc` 优化 java-tron 的内存分配。方法如下：

首先安装 `tcmalloc`，然后在启动脚本中添加以下两行，不同 Linux 发行版的 `tcmalloc` 路径略有不同。

```bash
#!/bin/bash

export LD_PRELOAD="/usr/lib/libtcmalloc.so.4"
export TCMALLOC_RELEASE_RATE=10

# original start command
java -jar .....
```

各个 Linux 发行版的说明如下：

- **Ubuntu 20.04 LTS / Ubuntu 18.04 LTS / Debian 稳定版**  
    安装

    ```shell
    $ sudo apt install libgoogle-perftools4
    ```

    在启动脚本中添加以下内容：

    ```bash
    export LD_PRELOAD="/usr/lib/x86_64-linux-gnu/libtcmalloc.so.4"
    export TCMALLOC_RELEASE_RATE=10
    ```

- **Ubuntu 16.04 LTS**  
    安装命令与上面相同。在启动脚本中添加以下内容：

    ```bash
    export LD_PRELOAD="/usr/lib/libtcmalloc.so.4"
    export TCMALLOC_RELEASE_RATE=10
    ```

- **CentOS 7**  
    安装

    ```shell
    $ sudo yum install gperftools-libs
    ```

    在启动脚本中添加以下内容：

    ```bash
    export LD_PRELOAD="/usr/lib64/libtcmalloc.so.4"
    export TCMALLOC_RELEASE_RATE=10
    ```