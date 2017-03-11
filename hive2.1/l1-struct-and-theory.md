## Hive架构和基本原理

### Hive和HDFS的关系

* HDFS 文件存储
* Yarn (MR, Tez) ExecDriver, SerDe
* MetaStore Parser, Semantic Analyzer, Optimizer

### Hive Compiler流程

Hive QL $\to (parser)$ Abstract Sentiment Tree $\to Sentiment\ analyzer $  Query Block $\to Logical\ plan\ generator$ Operator Tree $\to Logical\ optimizer$ Operator Tree $\to Physical\ plan\ generator$ Task Tree $\to Physical\ Optimizer$ Task Tree

大概的流程是，输入的Hive命令会变为String，然后通过Parser变成AST，AST会被分析成为QB (有两个部分，QB meta和?)，QB会生成逻辑执行的有向无环图并被优化 (各个Operator的执行顺序)，最后会生成能被MapReduce(或其他计算框)执行的有向无环图

可以通过`EXPLAIN`来看具体的执行过程

### 逻辑和执行阶段的优化

* 扫描相关

  * 谓词下推: 比如两张大表合并，然后选择，我们可以先对第一张大表做选择，然后在合并，这样大量减少了扫描数据的时间
  * 列剪裁: 只关系需要的列，比如下表中有a,b,c,d,e五列，下面的sql只会扫描a,b,e三列，默认`hive.optimize.cp=true`

  ```sql
  SELECT a, b from src where e < 10
  ```

  ​

  * 分区剪裁: 通过`WHERE`确定分区，默认`hive.optimize.pruner=true`

* 关联JOIN相关

  * Join操作左边为小表: Join操作的Reduce阶段，位于Join操作左边的表的内容会被加载进内存，小表放左边减少OOM错误
  * Join启动的job个数: 最后都会合并为一个Map-Reduce???
  * MapJoin: 表小于100mb，操作在Map阶段完成，前提是数据在Map阶段可以访问到
  * Join不支持不等值连接，因为是按照key分发，当有不等值时，会调用其他节点的key，hive是不支持的???

* 分组GROUP BY相关

  * Skew in data: 如果数据倾斜，一个reduce对应到的key的值太多，会影响hive运行的效率(等待极少数的reduce job)，`hive.groupby.skewindata=true`，当选项设定为true时，生成的查询计划会有两个MRjob。第一个map的结果集合会随机分布到reduce中，每个reduce做部分聚合操作，并输出结果，这样处理的结果是相同的group by key可能被分发到不同的reduce中，达到负载均衡；第二个map再根据预处理的数据结果按照group by key分布到reduce中(相同的group by key 在同一reduce中)，完成最终的操作

* 合并小文件

  * 对每个Hive任务增加一次mapreduce
  * 如何操作？？
  * 小文件由select where产生？？

### Hive 数据模型

DataBase, Table, Partition, Buckets (根据哈希值切分数据，目的是为了并行，在partition目录中，每个bucket对应一个文件), External Table (指向HDFS中存在的数据，CREATE TABLE的本质是mv，将HDFS文件移动到指定的`hive.metastore.warehouse.dir`中，CREATE EXTERNAL TABLE的本质仅是**指向**)

### 排序与分发的各种by

Hive与传统关系型数据库最大的区别是处理数据的能力，这种能力体现在排序与分发原理上.

* `ORDER BY`全部排序，只有一个reduce，最好不要使用
* `SORT BY` 随机分发到一个reduce然后reduce内部排序
* `DISTRIBUTE BY` 根据字段把对应的记录分发到那个reduce
* `CLUSTER BY`是`DISTRIBUTE BY + SORT BY`的缩写

### 琐碎笔记

* Spark SQL对复杂数据类型的支持并不好，比如Array, Map, Struct
* HQL，多看hive wiki上的Language Manual
* 通过自定义SerDe (RegexSerDe)，自定义正则，对文件解析