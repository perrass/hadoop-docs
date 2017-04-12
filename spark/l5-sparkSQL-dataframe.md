## Spark SQL与DataFrame

Spark结构

* Spark Core (RDD) 最底层
* 上层是Spark SQL和Catalyst (Spark SQL优化器)
* Catalyst上层是SQL优化和DataFrame/Dataset
* DataFrame/Dataset上层是ML Piplines，Structured Streaming和Graph Frames

DataFrame的可以实现编译时报错(`df.select("id")`)，而用SQL查询时是运行时报错，所以DataFrame更友好