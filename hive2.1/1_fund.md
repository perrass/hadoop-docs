Hive原理及查询优化（公开课视频）
===

### 基本

* Hive的最小处理单元是Operators(用于集群的Map, Reduce任务或者HDFS文件操作)
* Hive的Compiler的作用，是把一条Hive SQL，转化为一个Operators图（大部分情况是树）
