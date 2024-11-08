# 主网数据库快照
***

波场官方定期提供数据库快照以便快速部署节点。
数据快照是波场网络节点在特定时间的数据库备份的压缩文件。
开发者可以下载并使用这些数据快照来加速节点的同步过程。

## 下载数据快照

### 全节点数据快照
下表显示了全节点数据快照的下载地址。请根据位置、节点数据库类型以及是否需要查询历史内部交易，选择合适的数据快照。

| 全节点数据源 | 下载地址 | 描述 |
| ------------ | -------- | ---- |
| 官方数据源 (亚洲: 新加坡) | http://34.143.247.77 | LevelDB，不包含内部交易（2024年5月16日约1706G） |
| 官方数据源 (美洲) | http://34.86.86.229 | LevelDB，不包含内部交易（2024年5月14日约1702G） |
| 官方数据源 (亚洲: 新加坡) | http://35.197.17.205 | RocksDB，不包含内部交易（2024年5月16日约1686G） |
| 官方数据源 (新加坡) | http://35.247.128.170 | LevelDB，包含内部交易（2024年5月16日约1884G） |
| 官方数据源 (包含账户余额) | http://34.48.6.163 | LevelDB，不包含内部交易，但包含地址历史 TRX 余额（2024年5月16日约2143G） |

!!! note
    LevelDB 和 RocksDB 的数据不允许混合使用，数据库可在全节点的配置文件中指定，将 db.engine 设置为 LEVELDB 或 ROCKSDB。

### 轻量全节点数据快照
自 GreatVoyage-v4.1.0 版本发布以来，波场公链已支持轻量全节点类型。轻量全节点运行所需的数据是完整的状态数据和少量必要的区块数据，因此比普通全节点更轻量（数据库更小，启动更快）。波场官方提供轻量全节点的数据库快照。

| 轻量全节点数据源 | 下载地址 | 描述 |
| ---------------- | -------- | ---- |
| 官方数据源 (北美洲: 弗吉尼亚) | http://34.143.247.77/ | LevelDB |

!!! tip
    可以使用轻量全节点工具从整套数据中分离出所需数据。

## 使用数据快照
使用数据快照的步骤如下：

1. 根据需要下载相应的压缩备份数据库。
2. 解压缩备份数据库的压缩文件到 output-directory 目录，或根据需要解压到相应的目录。
3. 启动节点。节点默认读取 output-directory 目录。如果需要指定其他目录，请在节点启动时添加 -d directory 参数。

## 其他

### 轻量全节点工具
轻量全节点工具用于将全节点的数据库拆分为快照数据集和历史数据集。

- `快照数据集`: 轻量全节点快速启动所需的最小数据集。
- `历史数据集`: 用于历史数据查询的归档数据集。

在使用该工具进行任何操作之前，需要先停止当前运行的全节点进程。
此工具根据当前最新区块高度（`latest_block_number`）提供将完整数据分为两个数据集的功能。
从快照数据集中启动的轻量全节点不支持查询此区块高度之前的历史数据。该工具还提供将历史数据集与快照数据集合并的功能。

欲了解更多设计细节，请参阅: [TIP-128](https://github.com/tronprotocol/tips/issues/128)。

**获取轻量全节点工具**

可以通过编译 java-tron 源代码获得 LiteFullNodeTool.jar，步骤如下：

1. 获取 java-tron 源代码

    ```bash
    $ git clone https://github.com/tronprotocol/java-tron.git
    $ git checkout -t origin/master
    ```

2. 编译

    ```bash
    $ cd java-tron
    $ ./gradlew clean build -x test
    ```

    编译后，`LiteFullNodeTool.jar` 将生成在 `java-tron/build/libs/` 目录下。

**使用轻量全节点工具**  

**可选**

该工具提供独立的快照数据集和历史数据集的切割及合并功能。

- `--operation | -o`: `[ split | merge ]` 指定操作为切割或合并。
- `--type | -t`: `[ snapshot | history ]` 仅在切割时使用，指定要切割的数据集类型；`snapshot` 指快照数据集，`history` 指历史数据集。
- `--fn-data-path`: 全节点数据库目录。
- `--dataset-path`: 数据集目录，当操作为切割时，`dataset-path` 是存储快照数据集或历史数据集的路径，否则 `dataset-path` 应为历史数据集路径。

**示例**  

使用默认配置启动一个新全节点，然后在当前目录下生成 `output-directory`。
`output-directory` 包含一个名为 `database` 的子目录，该目录即为待切割的数据库。

- **切割并获取快照数据集**

    首先，停止全节点并执行：

    ```bash
    // 为简化起见，将快照定位在 `/tmp` 目录，
    $ java -jar LiteFullNodeTool.jar -o split -t snapshot --fn-data-path output-directory/database --dataset-path /tmp
    ```

    然后将会在 `/tmp` 生成一个 `snapshot` 目录，打包此目录并复制到准备运行轻量全节点的地方。不要忘记将此目录重命名为 `database`。
    
    !!! note
        `storage.db.directory` 的默认值为 `database`，确保将快照重命名为指定值


- **切割并获取历史数据集**

    如果需要历史数据查询，需要生成历史数据集并将其合并到轻量全节点。

    ```bash
    // 为简化起见，将历史定位在 `/tmp` 目录，
    $ java -jar LiteFullNodeTool.jar -o split -t history --fn-data-path output-directory/database --dataset-path /tmp
    ```

    一个 `history` 目录将会在 `/tmp` 生成，打包此目录并复制到轻量全节点。
    历史数据集通常占用较大存储空间，请确保磁盘有足够容量存储历史数据集。

- **合并历史数据集和快照数据集**

    历史数据集和快照数据集都拥有一个 `info.properties` 文件，用于标识其分割的区块高度。
    确保历史数据集中的 `split_block_num` 不小于快照数据集中的相应值。

    获取历史数据集后，轻量全节点可以合并历史数据集，成为一个真正的全节点。

    ```bash
    // 为简化起见，假设 `历史数据集` 位于 /tmp
    $ java -jar LiteFullNodeTool.jar -o merge --fn-data-path output-directory/database --dataset-path /tmp/history
    ```