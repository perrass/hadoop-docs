## Spark基础入门

### MapReduce局限性

* 仅支持Map和Reduce两种操作
* 处理效率低
  * Map中间结果写磁盘，Reduce写HDFS，多个MR之间通过HDFS交换数据
  * 任务调度和启动开销大
  * 无法充分利用内存
  * Map端和Reduce端均需要排序**?**
* 不适合迭代计算，交互式处理，流失处理
* 变成不够灵活

### Spark提升的地方

* 内存计算引擎，提供cache机制来支持需要反复迭代计算或者多次数据共享，减少数据读取的IO开销
* DAG引擎(其实本质多个MapReduce也是DAG)，减少多次计算之间中间结果写到HDFS的开销
* 使用多线程池模型来减少task启动开销，shuffle过程中避免不必要的sort操作以及减少磁盘IO操作

---

```sql
SELECT a.state, COUNT(*), AVERAGE(c.price) FROM a
JOIN b on a.id = b.id
JOIN c on a.itemId = c.itemId
GROUP BY a.state
```

如果使用Hive，则会有四个MR，分别是**????**

如果使用Spark SQL，或者Hive On Spark，则只是一个DAG图，没有MR的IO开销

---

Spark Cache将RDD缓存到内存中或磁盘上

```scala
val NONE = new StorageLevel(false, false, false, false)
val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
```

`data.cache()`实际上是`data.persist(StorageLevel.DISK_ONLY_2)`

### 提交Spark程序

```shell
spark-submit \
	--master yarn-cluster \ # 运行模式
	--class com.hulu.examples.SparkPi \  # main函数的入口
	--name sparkpi \ # 作业名称
	--driver-memory 2g \  # Driver需要的内存
	--driver-core 1 \  # Driver数量
	--executor-memory 3g \  # 每个executor需要的内存
	--executor-cores 3 \  # 每个executor线程数
	--num-executor 2 \  # 需启动的executor数量
	--queue spark \  # 提交到的队列
	--conf spark.pi.iterators=500000 \
	$FWDIR/target/scala-2.10/spark-example-assembly-1.0.jar
```

这个方式会上传spark-example-assembly-1.0.jar到集群，启动速度会较慢。这种情况下，**一个driver会有2个executor，每个executor有3个线程

