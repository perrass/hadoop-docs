# Apach Spark 源码分析

## Spark整体框架

Spark的计算过程是Input Data经过多个Operation变为Output Data的过程，这个过程的载体的RDD (Resilient Distributed Dataset)弹性分布式数据集，其特点是:

* 数据全集被分割为多个正相交的数据子集，每个数据子集可以被派发到任一计算节点进行处理
* 中间结果被保存，同一个计算结果被保存于多个计算节点
* 如果其中某一数据子集在处理中出现问题，针对该数据子集的处理会被重新调度进而重新处理

这个过程的Operation分为Transformation和Action，对应scala的`lazy val`

当Spark接收到提交的作业后，会进行如下处理

1. RDD之间的依赖性分析，这种依赖形成一个有向无环图DAG，**依赖关系的分析和判断由DAGScheduler负责**
2. 根据DAG的分析结果，将一个作业分成多个Stage。划分Stage的一个主要依据是当前的计算因子输入是否确定，如果是则分在同一个Stage之中
3. DAGScheduler确定完Stage之后，会向TaskScheduler提交任务集(Taskset)，而TaskScheduler负责将这些任务一一分发到集群的计算节点（Executor）

## SparkContext初始化

SparkContext是进行Spark应用开发的主要接口，是Spark上层应用与底层实现的中转站，其初始化的过程中涉及到：

* SparkEnv
* DAGScheduler
* TaskScheduler
* SchedulerBackend
* WebUI

步骤如下：

1. 根据初始化参数生成SparkConf，再根据SparkConf来创建SparkEnv。SparkEnv中主要包含以下关键组件：BlockManager, MapOutputTracker, ShuffleFetcher, ConnectionManager

```scala
private[spark] val env = SparkEnv.create(
	conf,
  	"",
  	conf.get("spark.driver.host"),
  	conf.get("spark.driver.port").toInt, 
  	isDriver = true,
  	isLocal = isLocal
)
SparkEnv.set(env)
```

2. 创建TaskScheduler。通过createTaskScheduler来判断Spark当前的部署方式，进而选择相应的SchedulerBackend的不同子类，启动TaskScheduler

```scala
private[spark] var taskScheduler = SparkContext.createTaskScheduler(this, master, appName)
taskScheduler.start()  //启动相应的SchedulerBackend，并启动定时器进行检测
```

3. 以2中TaskScheduler为实例为入参创建DAGScheduler并运行

```scala
@volatile private[spark] var dagScheduler = new DAGScheduler(taskScheduler)
DAGScheduler.start()
```

4. 启动WebUI

```scala
ui.start()
```

## 作业提交

## 作业执行

## 存储机制

## Spark SQL

## GraphX

基于Pregel这个基本的图计算处理模型实现的，因此Spark原生态地支持类Pregel接口及操作。

图具有多种类型的存储结构，常见的包括邻接矩阵，邻接表，十字链表。在Spark中，采用的是基于RDD的数据存储和设计

图数据本身具有连通性和图计算表现出来的强耦合性，因此，在并行计算的场景下，需要利用有效的图分割进行解耦。将一个大图分割为若干子图，有两个主要原则

* 提高子图内部的连通性，降低子图之间的连通性
* 考虑子图规模的均衡性，尽量保证各子图的数据规模均衡，不要出现较大的偏斜

常见的分割方式为：边分割和点分割

## MLlib