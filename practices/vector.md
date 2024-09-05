# MAXIR向量版最佳实践

## 1. 访问

MAXIR 提供了多种访问集群的方式，以满足不同的用户需求。本文涉及文件导入操作，建议使用方式一操作。

**方式一：使用 postgres client psql 访问**

在与MAXIR处于同一VPC下的云主机上，安装postgres client psql 版本。建议最佳为12版本。用户名操作请参考 [用户管理](/maxir/guides/dw-users/manage-dwusers) 。

```plain
psql -h {maxir地址} -U {maxir控制台创建的用户名} -d ${maxir_database}
```

**方式二：使用云平台提供的“DWSU控制台”访问**

如您想详细了解如何创建集群，请参考 [访问集群](/maxir/guides/dps-clusters/access-dps-clusters.md#访问-dps-集群) 。


## 2. DDL

### 2.1 CREATE TABLE

#### 示例

```postgresql
CREATE TABLE test_tbl
(
  id                  VARCHAR,
  vector              vecf16(25),
  post_publish_time   TIMESTAMP
)
USING heap distributed by(id)
PARTITION BY RANGE (post_publish_time) (START (DATE '2024-08-01') INCLUSIVE END (DATE '2024-08-31') EXCLUSIVE EVERY(INTERVAL '1 day'), DEFAULT PARTITION extra);
ALTER TABLE test_tbl ALTER COLUMN vector SET STORAGE PLAIN;
```

创建以后表的信息如下图：

![](/images/guides/optimization/practice.pic.jpg)


* 表数据存储依据id作分布

* 使用时间标识的列（创建时间，插入时间等）作为分区列，按天创建分区表用于更高效的增量更新数据

* 修改vector（向量数据）的存储方式为plain以加快查询

#### 向量类型介绍

| **类型名称** | **描述**                  |
| -------- | ----------------------- |
| vecf16   | 向量半精度数据，数据类型为float16浮点数 |

### 2.2 CREATE INDEX

#### 示例

```postgresql
CREATE INDEX test_vector_idx ON test_tbl USING vectors (vector vecf16_l2_ops) WITH (options='
optimizing.optimizing_threads = 5
segment.max_growing_segment_size = 100000
segment.max_sealed_segment_size = 8000000
[indexing.hnsw]
m=30
ef_construction=250
quantization.product.ratio = "x16"
');
```

增加索引后的表结构：

![](/images/guides/optimization/practice2.pic.jpg)

#### 参数介绍

##### vecf16\_l2\_ops

vecf16对应建表语句中向量列的类型

l2\_ops表示向量的距离计算公式

向量距离计算支持以下3种：

| 类型名称    | 操作符(可在SELECT中使用该操作符计算距离，详见下方查询语句) | 描述     | 计算公式定义                                                                                                                      |
| ------- | --------------------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------- |
| l2\_ops | `<->`                             | 平方欧氏距离 | ![](/images/guides/optimization/1-1.png)|


**建议使用l2\_ops以达到最佳性能**

##### optimizing.optimizing\_threads = 5

##### segment.max\_growing\_segment\_size = 100000

##### segment.max\_sealed\_segment\_size = 8000000

以上部分为基础向量索引参数，其取值为实际测试过程中获取的最佳值，可直接使用。

其具体含义如下：

| 键                                  | 类型        | 取值范围                    | 默认值         | 说明              |
| ---------------------------------- | --------- | ----------------------- | ----------- | --------------- |
| `optimizing.optimizing_threads`    | `integer` | \[1, 65535\]            | `1`         | 构建索引的最大线程数。     |
| `optimizing.sealing_secs`          | `integer` | \[1, 60\]               | `60`        | 构建索引合并探测时间      |
| `segment.max_growing_segment_size` | `integer` | \[1, 4\_000\_000\_000\] | `20_000`    | 未创建索引的向量的最大大小。  |
| `segment.max_sealed_segment_size`  | `integer` | \[1, 4\_000\_000\_000\] | `1_000_000` | 用于索引创建的向量的最大大小。 |

##### \[indexing.hnsw\]

##### m=30

##### ef\_construction=250

##### quantization.product.ratio = "x16"

以上部分指定向量索引的具体类型，以及该类型的一些相关配置。

选择适当的索引算法对于向量搜索的效率、资源利用率以及准确性等至关重要。MAXIR 支持如下三种算法：

##### HNSW（Hierarchical Navigable Small World）

HNSW算法将跳跃列表（skip list）的概念与可导航小世界图相结合，通过采用分层结构实现了高效的近似最近邻搜索。

HNSW 算法适用于大规模数据集和高维度数据的近似最近邻搜索任务。其优势在于能够快速导航和搜索，并且具有较好的搜索准确性。与传统的 NSW 搜索相比，HNSW 在搜索过程中利用了分层结构，避免了陷入局部最优的问题，提高了搜索的效率和准确性。

其中指定的hnsw索引相关参数含义如下：

| 键                 | 类型        | 取值范围                                          | 默认值   | 说明                     |
| ----------------- | --------- | --------------------------------------------- | ----- | ---------------------- |
| `m`               | `integer` | \[4, 128\]                                    | `12`  | 节点的最大度数。               |
| `ef_construction` | `integer` | \[10, 2000\]                                  | `300` | 构建中的搜索范围。              |
| `quantization`    | `table`   | trivial:不使用量化 scalar:使用标量量 product:使用乘积量化(推荐) | N/A   | 计算距离使用的量化算法，其具体参数见下方表格 |

##### `quantization.product` 的选项含义

| 键        | 类型        | 取值范围                            | 默认值     | 说明       |
| -------- | --------- | ------------------------------- | ------- | -------- |
| `sample` | `integer` | \[1, 1\_000\_000\]              | `65535` | 用于量化的样本。 |
| `ratio`  | `enum`    | "x4", "x8", "x16", "x32", "x64" | `"x4"`  | 量化的压缩比。  |

## 3. LOAD DATA

### 3.1 生成导入数据

测试数据可由[https://huggingface.co/datasets/hhy3/ann-datasets/tree/main](https://huggingface.co/datasets/hhy3/ann-datasets/tree/main)下载

本文选取glove-25-angular作为实践数据集

通过如下python脚本可以提取出数据集中的训练数据集（glove-25-angular-train.csv）

> 若不存在pandas或h5py，请通过以下命令安装:
> 
> pip3 install pandas
> python3 -m pip install h5py

```python
import uuid
import random
import pandas as pd
import h5py

# 读取 hdf5 文件
file = h5py.File('glove-25-angular.hdf5', 'r')

print("output train!")
df = pd.DataFrame(file['train'][:])
df['ID'] = [str(uuid.uuid4()) for _ in range(len(df))]
date_range = pd.date_range(start='2024-08-01', end='2024-08-31', freq='T')
df['Timestamp'] = [random.choice(date_range).isoformat() for _ in range(len(df))]
df['Data'] = df.apply(lambda row: '[' + ','.join(map(str, row[:-2])) + ']', axis=1)

# 调整列顺序
df = df[['ID', 'Data', 'Timestamp']]

# 将结果写入新的 CSV 文件，使用 '|' 作为分隔符
df.to_csv('glove-25-angular-train.csv', sep='|', index=False, header=False)
```

提取出的数据每一行为一条数据，数据格式如下，分别对应ID，向量数据及publish\_time，用｜分开

```postgresql
b6b07b6d-7f6c-4ffc-b2c5-f753cefc2e90|[-0.28571999073028564,1.6030000448226929,-0.23368999361991882,0.42476001381874084,0.071834996342659,-1.6633000373840332,-0.6774700284004211,-0.20066000521183014,0.7255899906158447,-0.722599983215332,0.09668300300836563,1.0442999601364136,1.1964000463485718,-0.2735399901866913,1.44159996509552,0.06502100080251694,0.9345399737358093,-0.40575000643730164,0.9226999878883362,-0.29600998759269714,-0.5180299878120422,0.8512099981307983,-1.0339000225067139,0.050655998289585114,0.13964000344276428]|2024-08-16T07:22:00
```

数据上传到us3（us3使用文档参考[https://docs.ucloud.cn/ufile/tools/us3cli/introduction](https://docs.ucloud.cn/ufile/tools/us3cli/introduction)）

```postgresql
us3cli cp glove-25-angular-train.csv us3://maxir-data-001/backup-data/glove-25-angular-train.csv
```

### 3.2 本地导入

SQL语句默认无法从本地文件导入数据，若需要导入客户端所在机器的数据文件，可以使用psql客户端进行连接，示例：

假设路径为/home/postgres/glove-25-angular-train.csv，使用psql进行连接（连接地址及账号密码可从控制台中获取）:

```sql
psql -h xxxx -p xxxx -d xxxx 
```

然后执行下面的语句：

```postgresql
\COPY test_tbl from '/home/postgres/glove-25-angular-train.csv'  WITH CSV DELIMITER '|';
```

### 3.3 从us3导入

数据已经正常上传到us3上，获取us3的路径和ak，sk（参考文档：[https://docs.ucloud.cn/ufile/quick/quick\_start](https://docs.ucloud.cn/ufile/quick/quick_start)）：

```postgresql
copy test_tbl from 's3://internal.s3-cn-sh2.ufileos.com/maxir-data-001/backup-data/glove-25-angular-train.csv'
ACCESS_KEY_ID 'xxxxxxxx'
SECRET_ACCESS_KEY 'xxxxxxxx'
DELIMITER '|' CSV;
```

## 4. QUERY

**q1: 查询所有向量中离目标向量最近的10个向量** 

```postgresql
SELECT id,
vector <-> '[0.6377800107002258,0.9509999752044678,0.9408400058746338,-0.5509499907493591,0.06180400028824806,-1.6734999418258667,-0.5704600214958191,-1.5750000476837158,0.5274199843406677,-0.3642300069332123,0.5622000098228455,0.009283199906349182,0.391759991645813,0.46647000312805176,-0.7589899897575378,0.3084399998188019,0.4611699879169464,0.30028998851776123,1.5491000413894653,1.2386000156402588,-0.7254599928855896,1.7488000392913818,0.4075799882411957,-1.96589994430542,0.05322200059890747]'
AS dist FROM test_tbl order by dist limit 10;
```

**q2：查询8月8号到15号区间内，由公式 dist \* 10 为算法计算相似度，获得相似度大于67，最相似的20条数据**

```postgresql
SELECT b.* FROM (SELECT a.id, a.dist * 10 AS similarity FROM
(SELECT id,
vector <-> '[0.6377800107002258,0.9509999752044678,0.9408400058746338,-0.5509499907493591,0.06180400028824806,-1.6734999418258667,-0.5704600214958191,-1.5750000476837158,0.5274199843406677,-0.3642300069332123,0.5622000098228455,0.009283199906349182,0.391759991645813,0.46647000312805176,-0.7589899897575378,0.3084399998188019,0.4611699879169464,0.30028998851776123,1.5491000413894653,1.2386000156402588,-0.7254599928855896,1.7488000392913818,0.4075799882411957,-1.96589994430542,0.05322200059890747]'
AS dist FROM test_tbl
WHERE post_publish_time >= '2024-08-08 00:00:00'
AND post_publish_time <= '2024-8-15 10:52:00'
ORDER BY dist ASC LIMIT 5000) as a) as b  WHERE b.similarity > 67  offset 0 limit 20;
```

## 5. 使用python客户端连接MAXIR

### 5.1 下载依赖python包

`psycopg2`：用于与MAXIR交互

使用 `pip3 install psycopg2-binary` 自助安装psycopg2包

### 5.2 编写python程序执行

#### 查询

执行python脚本，并且把需要执行的sql放到sql\_template中即可执行脚本

```python
import psycopg2

database='postgres'
user='postgres'
password=''
host='localhost'
port='5432'


def run_query(table, vector):
    conn = psycopg2.connect(database=database, user=user,
                            password=password, host=host, port=port)

    sql_template = " select id from " + table + " order by vector <-> "
    cur = conn.cursor()

    sql = sql_template + "'" + vector + "'" + " limit 3;"
    cur.execute(sql)

    cur.close()
    conn.close()

if __name__ == '__main__':
  run_query('postgres', '[1,2,3]')
```

#### 插入数据

使用命令`python3 insert.py --t 10 --dir ./data --batch 128`执行数据的导入，需要导入的数据放到data目录中，执行脚本之前先确认好database，user，password，host等参数

```python
import argparse
import psycopg2
from multiprocessing import Process, Value, Lock, Event, Queue
import os
import time
import sys

database='postgres'
user='postgres'
password=''
host='localhost'
port='5432'
insert_init = "insert into test_tbl values "
g_max_retry=3
ignore_error=True
load_batch=128
num_threads=10
load_dir='data'

def execute_with_retry(conn, sql):
    failed = True
    for i in range(g_max_retry):
        try:
            cur = conn.cursor()
            cur.execute(sql)
            cur.close()
            conn.commit()
            failed = False
            break
        except psycopg2.DatabaseError as e:
            print(f"[WARNING] execute sql {sql[:100]} failed, error: {str(e)}")
            conn.rollback()

    return failed


def load_task():
    print("load:", load_dir)
    processes = []
    global num_threads

    stop_event = Event()
    file_queue = Queue()
    lines = Value('i', 0)
    lines_lock = Lock()

    files = os.listdir(load_dir)
    tmp_input_files = [os.path.join(load_dir, filename) for filename in files]
    # remove directory
    for path in tmp_input_files:
        if os.path.isdir(path):
            print(f"[WARNING] remove directory {path}")
        else:
            file_queue.put(path)
            print(f"[INFO] need load file {path}.")

    if file_queue.qsize() <= 0:
        print(f"[WARNING] do not have invalid file at dir {load_dir}, exit.")
        sys.exit

    input_file_size = file_queue.qsize()
    if num_threads > input_file_size:
        print(
            f"[WARNING] num_threads:{num_threads} > input_files len:{input_file_size}, we just need {input_file_size} threads.")
        num_threads = input_file_size

    start_time = time.time()
    for i in range(num_threads):
        process = Process(target=do_load, args=(stop_event, file_queue, lines, lines_lock,))
        process.start()
        processes.append(process)

    for p in processes:
        p.join()

    end_time = time.time()
    interval = end_time - start_time
    print(f"[INFO] RPS: {lines.value / interval}, load data time:{interval}, total lines {lines.value}.")


def do_load(stop_event, file_queue, lines, lines_lock):
    print("do_load")
    try:
        conn = psycopg2.connect(database=database, user=user,
                                password=password, host=host, port=port)

        while not file_queue.empty():
            filename = file_queue.get()
            total_cnt = 0

            try:
                cnt = 0
                sql = insert_init
                failed = False
                with open(filename, 'r') as f:
                    for line in f:
                        if stop_event.is_set():
                            print("stop by event, exit.")
                            failed = True
                            break

                        line = line.replace('|', "','")
                        line = line.strip()

                        if cnt >= (load_batch - 1):
                            sql = sql + "('" + line + "');"
                            # print(sql)
                            failed = execute_with_retry(conn, sql)
                            if failed and not ignore_error:
                                break

                            total_cnt += cnt
                            cnt = 0
                            sql = insert_init
                        else:
                            sql = sql + "('" + line + "'),"
                            cnt += 1

                if cnt > 0:
                    # print("last batch")
                    sql = sql[:-1] + ";"
                    # print(sql)
                    failed = execute_with_retry(conn, sql)
                    if not failed:
                        total_cnt += cnt

            except Exception as e:
                print(f"[FATAL] open file {filename} failed: {str(e)}")
                stop_event.set()
                failed = True

            print(f"[INFO] load {total_cnt} lines into database.")
            with lines_lock:
                lines.value += total_cnt

            if failed:
                print(f"[FATAL] execute sql faild, exit.")
                stop_event.set()
                break

        conn.close()
    except Exception as e:
        print(f"[FATAL] run do_load woker faild, exit. {e} ")


if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Process some flags.')
    parser.add_argument('--t', type=int, help='Number of threads')
    parser.add_argument('--dir', type=str, help='load data dir path')
    parser.add_argument('--batch', type=int, default=100, help='batch num')

    args = parser.parse_args()
    num_threads = args.t
    load_dir = args.dir
    load_batch = args.batch

    load_task()
```

## 6. 使用pyspark客户端连接MAXIR

### 6.1 下载依赖python包

- `psycopg2`：用于与MAXIR交互

```shell
yum install postgresql-devel*
pip3 install psycopg2-binary
```

- `pyspark`：用于使用spark客户端

```shell
pip3 install pyspark
```

### 6.2 编写pyspark程序执行

#### 准备工作

- **测试数据文件 file.csv 内容格式参考：（测试时可手动复制多条）**

```csv
id,vector,post_publish_time
b6b07b6d-7f6c-4ffc-b2c5-f753cefc2e90,"-0.28571999073028564,1.6030000448226929,-0.23368999361991882,0.42476001381874084,0.071834996342659,-1.6633000373840332,-0.6774700284004211,-0.20066000521183014,0.7255899906158447,-0.722599983215332,0.09668300300836563,1.0442999601364136,1.1964000463485718,-0.2735399901866913,1.44159996509552,0.06502100080251694,0.9345399737358093,-0.40575000643730164,0.9226999878883362,-0.29600998759269714,-0.5180299878120422,0.8512099981307983,-1.0339000225067139,0.050655998289585114,0.13964000344276428",2024-08-16T07:22:00
```

1. 测试前需要手动在 MAXIR 上建好 database，以及 table，参考步骤2 DDL章节。

2. 本代码基于 hdfs，需要先将数据文件（file.csv） put 到 hdfs，可按需修改脚本中数据文件路径。

3. 需要下载postgresql-42.3.3.jar包：[postgresql-42.3.3.jar](https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar)，下载地址为：[https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar](https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar%E3%80%82)。

#### 通过pyspark插入数据

有两种方式写入数据到 MAXIR，分别为：

1. Load csv 文件方式：COPY ... FROM ...

2. JDBC 写入方式

##### Load csv 文件方式

通过提交本地spark任务，将数据通过spark预处理后导入到MAXIR中，提交任务命令：

```shell
spark-submit --deploy-mode client copydatatoMAXIR.py
```

- copydatatoMAXIR.py

```python
import psycopg2
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf, col
from pyspark.sql.types import StringType
import os
import glob
import subprocess
import shutil

# 创建SparkSession
spark = SparkSession.builder \
    .appName("File splitter and copy to MAXIR") \
    .getOrCreate()

# 读取本地文件
input_path = "/user/hadoop/spark-maxir-demo/file.csv"
df = spark.read.csv(input_path, header=True, inferSchema=True)

# 对向量数据预处理udf
def array_to_string(array):
    res = "[" + str(array) + "]"
    return res

array_to_string_udf = udf(array_to_string, StringType())

# 使用UDF对向量列进行转换
df_transformed = df.withColumn("vector", array_to_string_udf(col("vector")))

# 重新分区（此处测试demo，只设置1个分区，实际场景可按数据量进行分区）
df = df_transformed.repartition(1)

# 输出到本地
output_path = "/user/hadoop/spark-maxir-demo/out/"
df.write.option("delimiter", "|").csv(output_path, header=True, mode="overwrite")

local_output_path = "/tmp/spark-maxir-demo/out/"

# 清空本地目录
if os.path.exists(local_output_path):
    shutil.rmtree(local_output_path)

# 确保本地目录存在
os.makedirs(local_output_path, exist_ok=True)

# 下载 HDFS 上的文件到本地
subprocess.run(["hdfs", "dfs", "-get", output_path + "*", local_output_path])

# 连接到PostgreSQL数据库
conn = psycopg2.connect(
    dbname="dbname",
    user="user",
    password="password",
    host="127.0.0.1",
    port="5432"
)
cur = conn.cursor()

def find_csv_files(directory):
    # 使用glob模組查找所有以.csv結尾的文件
    csv_files = glob.glob(os.path.join(directory, "*.csv"))
    return csv_files

# 获取临时文件路径
csv_files = find_csv_files(local_output_path)
print("csv_files:", csv_files)

for csv_file in csv_files:
    print("csv_file:", csv_file)
    # 导入数据
    with open(csv_file, 'r') as f:
        print(f"pyspark:--------start copy {csv_file} to MAXIR--------")
        cur.copy_expert("COPY test_tbl (id, vector, post_publish_time) FROM stdin WITH (FORMAT csv, HEADER true, DELIMITER '|')", f)

# 提交事务并关闭连接
conn.commit()
cur.close()
conn.close()

# 停止SparkSession
spark.stop()
```

##### JDBC 写入方式

spark 任务提交命令：

```shell
spark-submit --deploy-mode client --jars postgresql-42.3.3.jar copydatatoMAXIR.py
```

- copydatatoMAXIR.py

```python
import psycopg2
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf, col
from pyspark.sql.types import StringType
import os
import glob
import subprocess
import shutil

# 创建SparkSession
spark = SparkSession.builder \
    .appName("File splitter and copy to MAXIR") \
    .getOrCreate()

# 读取本地文件
input_path = "/user/hadoop/spark-maxir-demo/file.csv"
df = spark.read.csv(input_path, header=True, inferSchema=True)

# 对向量数据预处理udf
def array_to_string(array):
    res = "[" + str(array) + "]"
    return res

array_to_string_udf = udf(array_to_string, StringType())

# 使用UDF对向量列进行转换
df_transformed = df.withColumn("vector", array_to_string_udf(col("vector")))

# 重新分区（此处测试demo，只设置1个分区，实际场景可按数据量进行分区）
df = df_transformed.repartition(1)

df.write \
    .mode("append") \
    .format("jdbc") \
    .option("url", "jdbc:postgresql://${ip}:5432/${database}") \
    .option("driver", "org.postgresql.Driver") \
    .option("isolationLevel", "REPEATABLE_READ") \
    .option("dbtable", "${tablename}") \
    .option("user", "${user}") \
    .option("password", "${}") \
    .save()

# 停止SparkSession
spark.stop()
```

#### 查询

通过提交本地spark任务，查询 MAXIR 中的数据，并且对查询的数据进行操作。提交任务命令：

```shell
spark-submit --deploy-mode client --jars postgresql-42.3.3.jar queryMAXIR.py
```

- queryMAXIR.py

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Read Postgres Data") \
    .getOrCreate()

# 设置Postgres数据库的JDBC URL
jdbc_url = "jdbc:postgresql://127.0.0.1:5432/postgres"

# 设置连接数据库
connection_properties = {
    "user": "user",
    "password": "password",
    "driver": "org.postgresql.Driver"
}

# 读取数据
df = spark.read.jdbc(url=jdbc_url, table="(SELECT id, vector <-> '[-0.28571999073028564,1.6030000448226929,-0.23368999361991882,0.42476001381874084,0.071834996342659,-1.6633000373840332,-0.6774700284004211,-0.20066000521183014,0.7255899906158447,-0.722599983215332,0.09668300300836563,1.0442999601364136,1.1964000463485718,-0.2735399901866913,1.44159996509552,0.06502100080251694,0.9345399737358093,-0.40575000643730164,0.9226999878883362,-0.29600998759269714,-0.5180299878120422,0.8512099981307983,-1.0339000225067139,0.050655998289585114,0.139640003442764285]' as distance FROM test_tbl order by distance) AS test_tbl_alias", properties=connection_properties)

# 显示数据
# +--------------------+--------+
# |                  id|distance|
# +--------------------+--------+
# |b6b07b6d-7f6c-4ff...|     0.0|
# |b6b07b6d-7f6c-4ff...|     0.0|
# +--------------------+--------+
df.show()

# 过滤数据
filtered_df = df.filter(df["distance"] == 0)

# 显示结果
# +--------------------+--------+
# |                  id|distance|
# +--------------------+--------+
# |b6b07b6d-7f6c-4ff...|     0.0|
# |b6b07b6d-7f6c-4ff...|     0.0|
# +--------------------+--------+
filtered_df.show()

spark.stop()
```
