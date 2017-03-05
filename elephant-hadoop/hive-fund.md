hive-fund
===

### Intro

HDFS存储，Yarn计算框架，MapReduce计算引擎

#### 边界

* Hive不是一个OLTP系统(实时请求)
    + 响应时间慢，用过Yarn
    + 无法实施更新数据，HDFS无法实时更新(不支持随机写，只能append)
* Hive的表达能力有限
    + 不支持迭代计算
    + 用SQL不易表达

### Hive Concepts

#### 数据模型

* **Database**:　和关系型一样, 用户管理和配额管理
* **Table**: 和关系型的表一样
* **Partition(分区)**: 对每个分区建一个目录　`.../employees/country=CA/state=AB`，利用这个目录减少不必要的暴力数据扫描，为避免产生过多小文件，建议**只对离散字段进行分区**
* **Bucket(分桶)**:　对于值较多的字段，将其分成若干个Bucket，比如淘宝的user_id
* **File**

    CREATE TABLE employees (
        name            STRING,
        salary          FLOAT,
        subordinates    ARRAY<STRING>,
        deductions      MAP<STRING, FLOAT>,
        address         STRUCT<street: STRING, city: STRING, state: STRING, zip: INT>
    )
    PARTITIONED BY (country STRING, state STRING)

    SELECT * FROM employees
    WHERE country = "US" AND state = "AL"

    CREATE TABLE weblog (user id INT, url STRING, source_ip STRING)
    PARTITIONED BY (dt STRING)
    CLUSTERED BY (user_id) INTO 96 BUCKETS;

#### 类型系统



#### 语言规范

### Hive DDL (Data Definition Laugage)

### Hive DML (Data Manipulate Laugage)
