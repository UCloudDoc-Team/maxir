# 通过 Flink 导入数据至 Relyt

本文展示了如何通过 Flink 将数据加载到 Relyt。

## 1. 准备工作

在开始之前，请确保您的环境中已经有一个 Flink 集群。如果没有，请按照本章节的指导完成 Flink 集群配置。如果您的环境中已经配置好了 Flink 集群，请跳过本章节直接进入[第二章](#2使用flink创建表)。

### 1.1 下载依赖包、工具包

1. 下载下述依赖包：
   
    Hadoop 依赖包：[https://archive.apache.org/dist/hadoop/common/hadoop-3.1.1/hadoop-3.1.1.tar.gz](https://archive.apache.org/dist/hadoop/common/hadoop-3.1.1/hadoop-3.1.1.tar.gz)

    Flink 依赖包：[https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala\_2.12.tgz](https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.12.tgz)

    对应版本的 Java 依赖包：[https://www.oracle.com/java/technologies/downloads/#java8](https://www.oracle.com/java/technologies/downloads/#java8)

    或者执行如下代码（此处以 `java-1.8.0-openjdk.x86_64` 为例）：

    ```shell
    yum -y list java*
    yum install java-1.8.0-openjdk.x86_64
    
    # 配置 Java 环境
    JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk.x86_64
    PATH=$PATH:$JAVA_HOME/bin
    CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.cdjar
    export JAVA_HOME CLASSPATH PATH
    :::

2.  下载 PostgreSQL JDBC 驱动器并保存至 Flink 的 `lib` 目录下：
    
    ```shell
    curl -O https://repo1.maven.org/maven2/org/postgresql/postgresql/42.2.5/postgresql-42.2.5.jar
    ```

3.  下载 Relyt CDC 驱动器并保存至 Flink 的 `lib` 目录下。
    

### 1.2 完成环境配置

1.  解压 Hadoop 压缩包。
    
2.  进入 Hadoop 目录，获取 Hadoop 的 classpath。
    
    ```shell
    # 请将 /usr/local/hadoop 替换为实际的 Hadoop 路径。
    cd /usr/local/hadoop
    bin/hadoop classpath
    ```

3.  在环境变量中，配置 Hadoop、Java 和 Flink 依赖：
    
    ```shell
    #set java environment
    JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
    PATH=$PATH:$JAVA_HOME/bin
    CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.cdjar
    export JAVA_HOME CLASSPATH PATH
    
    export HADOOP_CLASSPATH=/root/testonly/hadoop-3.1.0/etc/hadoop:/root/testonly/hadoop-3.1.0/share/hadoop/common/lib/*:/root/testonly/hadoop-3.1.0/share/hadoop/common/*:/root/testonly/hadoop-3.1.0/share/hadoop/hdfs:/root/testonly/hadoop-3.1.0/share/hadoop/hdfs/lib/*:/root/testonly/hadoop-3.1.0/share/hadoop/hdfs/*:/root/testonly/hadoop-3.1.0/share/hadoop/mapreduce/*:/root/testonly/hadoop-3.1.0/share/hadoop/yarn:/root/testonly/hadoop-3.1.0/share/hadoop/yarn/lib/*:/root/testonly/hadoop-3.1.0/share/hadoop/yarn/*
    export HADOOP_CONF_DIR=/root/testonly/hadoop-3.1.0/etc/hadoop
    ```

如上代码中 Hadoop 的 classpath 仅为示例。

4.  修改 Flink 配置文件 `conf/flink-conf.yaml`，配置 UI 及执行参数。完成配置后，执行如下命令启动集群：
    
    ```shell
    bin/start-cluster.sh
    ```

5.  执行如下命令打开 Flink 控制台，即可开始进行创建 Flink 表等相关 SQL 操作：
    
    ```shell
    bin/sql-client.sh
    ```

## 2. 使用 Flink 创建表

创建 Flink 源表和对应于 Relyt 的表。

示例 1：

```sql
CREATE TABLE flink_source (
    id INT,
    content VARCHAR
) WITH (
    'connector' = 'datagen',
    'rows-per-second' = 100,
    'number-of-rows' = 500
);
    
CREATE TABLE flink_to_relyt (
    id INT,
    content VARCHAR
) 
PARTITION BY (id)
WITH (
    'connector' = 'relyt',
    'url' = 'jdbc:postgresql://<host_name>:<port>/<db_name>',
    'username' = '<username>',
    'password' = '<password>',
    'table-name' = 'relyt_destination'
    'sink.basepath' = 'relyt://<bucket>/<prefix>',
    'sink.extra' = 'fs.relyt.endpoint=<s3_endpoint>,fs.relyt.access.key=<access_id>,fs.relyt.secret.key=<secret_key>',
    'parallelism'= <degree_of_parallelism>
);
```

```sql
-- Relyt 侧 table 建表
CREATE TABLE sink_tbl_JSON (
 "id" INT,
 "content" JSON
);


-- Flink 侧 sql 建表
CREATE TABLE sink_tbl_JSON (
 `id` INT, 
 `content` VARCHAR
)
WITH (
  'connector' = 'relyt',
  'url' = 'jdbc:postgresql://localhost:7800/postgres',
  'username' = 'postgres',
  'sink.basepath' = 'relyt://relyt-flink-cicd/tmpdata/',
  'sink.extra' = 'xxxxxxxxx',
  'table-name' = 'sink_tbl_JSON'
);

-- Flink sql 导入
INSERT INTO sink_tbl_JSON SELECT * FROM source;
```


如上示例代码创建了名为 `flink_source` 的 Flink 源表以及名为 `flink_to_relyt` 的对应于 Relyt 的表，请根据实际情况进行替换。其他相关参数说明，请参考下表：

|  参数  |  是否必选 |  说明  |  示例 |
| :- | :- | :- | :- |
|  connector  |  是  |  连接器名称。固定为 relyt。  |  `relyt`  |
|  url  |  是  |  Relyt 的连接地址。其中：*`<host_name>`* 为主机名，*`<port>`* 为端口号，*`<db_name>`* 为 Relyt 数据库名称。       |  `jdbc:postgresql://localhost:10001/postgres`  |
|  username  |  是  |  Relyt 连接用户名。  |  `test@example.com`  |
|  password  |  否  |  Relyt 连接密码。  |  `test123`  |
|  table-name  |  是  |  数据导入的目标 Relyt 表名，支持使用 Schema 名称进行限定。 |  `relyt_destination`  |
|  sink.basepath  |  是  |  数据导入的临时存储目录。必须以 `relyt://` 开头。  |  N/A  |
|  sink.extra  |  是  |  其他配置项，包括临时存储目录的 AK/SK、Endpoint、是否清理临时数据、并行度等。参数值支持键值对列表。 <br/>**说明：** <br/>当使用内网时，Endpoint 必须配置为 `internal`。  |  `fs.relyt.endpoint=cn-bj.ufileos.com,fs.relyt.aws.credentials.provider=org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider,fs.relyt.access.key=xxxxxx,fs.relyt.secret.key=yyyyyy,sink.clean_temp_data=false`  |
| parallelism | 是 | 并行度。 | `2` | 
|  sink.max_temp_file_size  |  否  |  临时目录最大支持的大小，单位为字节。默认为 96 MB。  |  `100663296`  |
|  sink.async_load_parallelism  |  否  |  异步导入任务并发数。  |  `2`  |
|  sink.async_load_queue_size  |  否  |  异步任务队列大小。当排队任务超过该值时，Flink 侧会等待 Relyt 侧完成当前任务后，再发起新的任务。  |  `4`  |
|  verbose  |  否  |  是否输出连接器的运行日志。 `false` 表示不输出运行日志，`true` 表示输出运行日志。       |  `false`  |


### 使用注意事项

Flink 表中的 Schema 需要和写入目标 Relyt 表的 Schema 一致，或者是目标表 Schema 的子集，且表中列的名称和类型需要和目标表保持一致。

关于 Flink 数据类型和 Relyt 数据类型对应关系，请参考下表：

|  Flink 数据类型  |  Relyt 数据类型  |
| :- | :- |
|  `CHAR`  |  `char`  |
|  `VARCHAR`  |  `varchar`  |
| `VARCHAR`  |  `json` (`jsonb`)  |
|  `STRING`  |  `varchar`  |
|  `BOOLEAN`  |  `boolean`  |
|  `BINARY`  |  不支持  |
|  `VARBINARY`  |  不支持  |
|  `BYTES`  |  不支持  |
|  `DECIMAL`  |  `decimal`  |
|  `TINYINT`  |  `smallint`  |
|  `SMALLINT`  |  `smallint`  |
|  `INTEGER`  |  `integer`  |
|  `BIGINT`  |  `bigint`  |
|  `FLOAT`  |  `float4`  |
|  `DOUBLE`  |  `float8`  |
|  `DATE`  |  `date`  |
|  `TIME`  |  不支持  |
|  `TIMESTAMP`  |  `timestamp`  |
|  `TIMESTAMP_LTZ`  |  `timestamp`  |
|  `INTERVAL`  |  不支持  |
|  `ARRAY`  |  不支持  |
|  `MULTISET`  |  不支持  |
|  `MAP`  |  不支持  |
|  `ROW`  |  不支持  |
|  `RAW`  |  不支持  |

<br/>

## 3. 写入数据至 Relyt

本节所有示例中均使用 `flink_to_relyt` 和 `flink_source` 表示对应于 Relyt 的 Flink 表名和 Flink 源表。正式使用时，请替换为实际表名。

**使用 `INSERT INTO` 进行写入：**

```sql
INSERT INTO flink_to_relyt SELECT * FROM flink_source;
```



**使用 `INSERT OVERWRITE` 进行全表覆写：**

```sql
INSERT OVERWRITE flink_to_relyt SELECT * FROM flink_source;
```


**使用 `INSERT OVERWRITE` 将表数据复制到特定分区：**

```sql
INSERT OVERWRITE flink_to_relyt PARTITION (id=<value>) SELECT * FROM flink_source;
```

:::warning `INSERT OVERWRITE` 的使用限制
- 只能用于处理 Flink 批任务。
- Flink 表中的分区列必须和 Relyt 表中的聚簇键相匹配。
- 支持的分区列类型包括 `CHAR`、`VARCHAR`、`TINYINT`、`SMALLINT`、`INTEGER`、`DATE`。关于 Flink 数据类型和 Relyt 数据类型对应关系，请参考 [使用注意事项](#使用注意事项) 中的数据类型映射表。
:::


## 4. 查看导入进展

您可以在 Flink 侧或者 Relyt 侧查看导入任务完成情况。

### 4.1 Flink 侧查看

您可以在 Flink 提供的 WebUI 上查看检查点 (Checkpoint) 的状态。一个检查点成功，意味着对应的数据已成功写入至临时存储目录，且在 Relyt 侧已完成任务元数据记录。

### 4.2 Relyt 侧查看

调用 `relyt_get_async_load_job()` 查看数据导入任务的状态：

```sql
SELECT * FROM relyt_get_async_load_job() WHERE target::regclass = 'relyt_destination'::regclass;
```

其中，`relyt_destination` 仅为示例，正式使用时，请替换为实际环境中的目标 Relyt 表。

可能的返回值包括：

- `INIT`：初始化中，数据正在写入临时存储目录中。
  
- `READY`：可执行，数据已写入至临时存储目录中。
  
- `LOADING`：数据导入中，数据正在从临时存储目录中导入至 Relyt。
  
- `FINISH`：任务已完成。
  
- `FAIL`：任务失败。
  
- `CANCEL`：任务已取消。
  

当返回值为 `FINISH` 时，导入任务已成功完成。
