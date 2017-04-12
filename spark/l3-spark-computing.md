## Spark计算引擎剖析

### 核心概念

#### Spark引擎内部执行流程

1. 生成逻辑查询计划（通过RDD，创建DAG）
2. 生成物理查询计划（为DAG生成物理查询计划）
3. 任务调度（调度并执行Task）
4. 任务执行（分布式执行Task）

或者

1. 用户提交Application
2. 按照Action的数量生成对应的Job
3. 每个Job对产生不同的Stage
4. 每个Stage会切分成多个task在executor上并行运行
   1. Task有ShuffleMapTask和ResultTask两类
   2. 如果是First Stage，task数由hdfs block或hbase region数决定；如果是Other Stage，由用户设置，默认与第一阶段相等

### 计算引擎原理

流程

1. RDD Object: `rdd1.join(rdd2).groupBy().filter()`，构造操作符DAG
2. DAGScheduler: 将图分解成一系列stage，每个stage由若干个task组成，将task提交到集群中
3. TaskScheduler: 通过cluster manager提交作业，
4. Executor: 执行task

#### 逻辑生成计划

RDD之间按照DAG图会有依赖关系，一种是FullDependency（一一或多一对应），一种是ShuffleDependency（多多对应）。SuffleDependency产生的连接数是$M \times R$ , M是map task，R是reduce task，Shuffle是分布式计算框架的核心数据交换方式，其实现方式直接决定了计算框架的性能和扩展性。在Spark中，**产生Shuffle的算子是join, cogroup和*ByKey**

ReduceByKey比GroupByKey多一步`ParallelCollectionRDD`到`MapPartitionRDD`，也就是一步按照key进行combine的过程，用来减少连接数

#### 物理执行计划

1. 划分Stage
2. 调度执行Task

**核心原则是**

* 按照Shuffle切分Stage
* 不同父依赖的Stage并行执行
* 每个Stage会切成不同Task，并行执行

#### 调度和提交任务

* 作业调度：FIFO或Fair
* 任务执行：
  * **Task被序列化后，发送到executor执行**
  * **ShuffleMapTask将中间结果写到本地磁盘，ResultTask运城读取数据**
  * **数据用的时候再算，且数据是流到要计算的位置** 

关于序列化

Java和Scala的对象不能通过直接通过网络传输，需要通过序列化转为字节流，经网络传输后再反序列化成为对象

关于Shuffle写本地磁盘

分布式计算的中间结果都会写本地磁盘，为了稳定性和容错，不写HDFS是因为读写本地磁盘的效率最高

### Shuffle解析

#### Write

* hash-based
  * 每个map task拆分为多个buckets (unsorted)，每个reduce task接受对应的buckets，这样总的shuffle file数量是$M \times R$
  * 如果在一个Core内，先对多个Map Tasks对应的buckets做汇总，在reduce，需要$Core \times R$
* sort-based
  * 每个map task产生一个file，这个file是sorted key-value pair，然后每个reduce task读取file内对应的标签的值

#### Read

hash-based和sort-based都用shuffle read来读，产生ShuffleRDD

#### Aggregate

unsorted partitioned key value pair -> key agg(value) pair

写的过程是hash-based，从unsorted records -> aggregate的过程是`map.put(key, f(value, map.get(key)))`