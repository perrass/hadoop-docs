## Spark编程实例

### Scala基本

#### 集合处理

```scala
var list = List(1, 2, 3)
list.foreach(x => println(x))
list.foreach(println)
  
list.map(x => x + 2)
list.map(_ + 2)
  
list.filter(x => x % 2 == 1)
list.filter(_ % 2 == 1)
  
list.reduce((x, y) => x + y)
list.reduce(_ + _)
```

`map`也可以传递函数，主要用于匿名函数太长的情况

```scala
def addTwo(x: Int): Int = x + 2
list.map(addTwo)
```

### Spark编程基础

#### Task设定

```scala
val slices = 10
val n = 100000 * slices	// 对每个task迭代10w次
val count = sc.parallelize(1 to n, slices)  // 并行度是slices的值（task的值)
  .map { i => 
       	val x = random * 2 - 1
       	val y = random * 2 - 1
       	if (x * x + y * y < 1) 1 else 0 }
  .reduce(_ + _)
```

#### 创建RDD

```scala
val inputRdd = sc.textFile("/data/input")  // 依照上下文确定读取什么文件
val inputRdd = sc.textFile("file:///data/input")  // 读取本地文件
val inputRdd = sc.textFile("hdfs:///data/input")  // 读取hdfs文件
val inputRdd = sc.textFile("hdfs://namenode:8020/data/input")  // ？？
```

读取HDFS数据时，HDFS有几个block，默认气氛几个partition。

#### Key/Value类型的RDD

```scala
val pets = sc.parallelize(List(("cat", 1), ("dog", 1), ("cat", 2)))
pets.reduceByKey(_+_)  // => {(cat, 3), (dog, 1)}
pets.groupByKey()  // => {(cat, Seq(1, 2)), (dog, Seq(1))} 结果是Seq类
pets.sortByKey()  // => {(cat, 1), (cat, 2), (dog, 1)}
```

所有key/value RDD操作符均包含并行度参数，控制reduce task.`words.reduceByKey(_ + _, 5)`，当并行度过多时用于控制，用于也可以通过修改`spark.default.parallelism`来设置

#### Accumulator

* 类似于MapReduce中的counter，将数据从一个节点发送到其他各节点上
* **用于监控，调试，记录符合某类特征的数据数**

```scala
import SparkContext._
val total_counter = sc.accumulator(0L, "total_counter")
val counter0 = sc.accumulator(0L, "counter0")
val counter1 = sc.accumulator(0L, "counter1")

val count = sc.parallelize(1 to n, slices)  // 并行度是slices的值（task的值)
  .map { i => 
       	val x = random * 2 - 1
       	val y = random * 2 - 1
       	if (x * x + y * y < 1) {
        	counter1 += 1
        } else { 
        	counter0 += 1
        }
        if (x * x + y * y < 1) 1 else 0
       }
  .reduce(_ + _)
```

#### Broadcast

* 高效分发大对象，比如字典，集合(set)，每个executor一份
* 包括HttpBroadcast和TorrentBroadcast

```scala
val data = Set(1, 2, 4, 6, ...)  // 大小为128MB
val rdd = sc.parallelize(1 to 6, 2)
val observedSizes = rdd.map(_ => map.size)
  
val bdata = sc.broadcast(data)
val rdd = sc.parallelize(1 to 1000000, 100)
val observedSizes = rdd.map(_ => bdata.value.size)  // 各个task中，通过bdata.value获取广播的集合
```

#### cache

当需要重复读取某变量时，数据从HDFS读取并缓存至内存（或者单独设立缓存机制），**如果只是用一次或次数极少，不需要用cache**

### Scala程序设计流程

1. 创建Maven项目
2. 用IDE做本地开发
3. Maven打包
4. 编写shell脚本运行程序（包括一些基本的命令行，如删除某个存在的目录）

```shell
mvn archetype:generate \ 
-DarchetypeGroupId=org.scala-tools.archetypes \ 
-DarchetypeArtifactId=scala-archetype-simple \
-DarchetypeVersion=1.1 \
-DremoteRepositories=http://scala-tools.org/repo-releases \
-DarchetypeCatalog=Internal \
-DinteractiveMode=false \
-Dversion=1.0-SNAPSHOT \
-DgroupId=org.training.spark \
-DartifactId=wordcount
```

```shell
mvn package
```

```shell
hdfs dfs -rm -r /xxx  # hdfs操作
spark-submit \  # 提交spark作业
--master
--class
/dir/xxx.jar  # 导入jar包
yarn-cluster /home/dir /tmp/dir  # args[]
```

### 例子

统计每个用户在每台机器上查询的次数(query)和返回结果累计大小(byte)

```scala
val apacheLogRegex = """^([\d.+)(\S+)(\S+)\[([\w\d:/]+\s[+\-]\d{4})\]"(.+?)"(\d{3})(\d{3})([\d\-]+)"([^"]+)"([^"]+)".*"""
def extractKey(line: String): (String, String, String) = {
  apacheLogRegex.findFirstIn(line) match {
    case Some(apacheLogRegex(ip, _, user, dataTime, query, status, bytes, referer, ua)) =>
      if (user != "\"-\"") (ip, user, query)
      else (null, null, null)
    case _ => (null, null, null)
  }
}
def extractStats(line: String): Stats = {
  apacheLogRegex.findFirstIn(line) match {
    case Some(apacheLogRegex(ip, _, user, dataTime, query, status, bytes, referer, ua)) => new Stats(1, bytes.toInt)
    case _ => new Stats(1, 0)  
  }
}
                                                                                                     
```

```scala
class Stats(val count: Int, val numBytes: Int) extends Serializable {
  def merge(other: Stats) = new Stats(count + other.count, numBytes + other.numBytes)
  override def toString = "bytes=%s\tn=%s".format(numBytes, count)
}
```

```scala
object LogQuery {
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("Spark Pi")
    val sc = new SparkContext(conf)
    val dataset = sc.textFile(args[0])
    dataSet.map(line => (extractKey(line), extractStats(line)))  // 形成KEY-VALUE PAIR
      .reduceByKey((a, b) => a.merge(b))  // 按Key做reduce，调用Stats.merge函数
      .collect().foreach {
        case (user, query) => println("%s\t%s".format(user, query))
      }
  }
}
```

