## 	Spark调优

### 运行环境优化

* 提高数据本地性（存储和计算在同一机器，减少网络开销）
* 存储格式ORC（Hive），Parquet（MR/Spark）

### 优化操作符

* 过滤操作导致很多小任务
* 降低单挑记录处理开销
* 处理数据倾斜或者任务倾斜
* 操作符选择
  * 避免使用cartesian
  * 尽可能避免shuffle
  * 如果可能用reduceByKey替代groupByKey
    * `rdd.groupByKey().mapValues(_.sum)`
    * `rdd.reduceByKey(_+_)`
  * 如果可能，用treeReduce代替reduce
  * 输入输出value类型不同时，避免使用reduceByKey
* 利用并行数控制shuffle
* 作业并行化，一个Spark应用程序由多个Job构成，如果Job之间没有依赖关系，可以并行处理
  * 启用FAIR调度器：`spark.scheduler.mode=fair`
  * 将action相关操作放到单独线程中

> 将key相同的value聚集到一起，并去重相同的value

```scala
rdd.map(kv => (kv._1, new Set[String]() + kv._2))
  .reduceByKey(_ ++ _)
  
//Better
val zero = new collection.mutable.Set[String]()
rdd.aggregateByKey(zero)(
	(set, v) => set += v,
	(set1, set2) => set1 ++= set2)
```

### 作业参数调优

