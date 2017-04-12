## 	Spark SQL程序设计与应用案例

```shell
mkdir /tmp/data
cat ml-1m/user.dat | tr -s "::" ',' >> /tmp/data/users.dat
spark-sql  # CDH没有spark-sql的CLI
```

```SQL
DROP TABLE user
CREATE EXTERNAL TABLE user (
	userid INT,
	gender String,
	age INT,
	occupation STRING,
	zipcode INT
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ","
STORED AS TEXTFILE
LOCATION 'tmp/data';
```

使用`hive-site.xml`拷贝到Spark安装包的conf目录下，可以使用Spark SQL处理Hive Metastore中的表

流程

1. 创建SQLContext对象
2. 创建DataFrame或DataSet
3. 在DataFrame或Dataset上进行transformation和action
4. 返回结果

```scala
import sqlContext.implicits._  // RDD隐式转换为DataFrame
  
val sc: SparkContext
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
```

### 创建DataFrame的方式

#### 1. RDD -> DataFrame：反射方式

* **定义case class, 做为RDD的schema**
* 直接通过RDD.toDF将RDD转换为DataFrame

```scala
import org.apache.spark.sql.SQLContext
import org.apache.spark.sql.Row
  
case class User(userID: Long, gender: String, age: Int, occupation: String, zipcode: Int)
    
val usersRdd = sc.textFile("/tmp/ml-1m/users.dat")
val userRDD = usersRdd.map(_.split(",")).map(p => User(p(0).toLong, p(1).trim, p(2).toInt, p(3), p(4).toInt))
val userDataFrame = userRDD.toDF()
```

#### 2. RDD -> DataFrame：显示注入Schema

* 定义RDD schema (由StructField/StructType构成)
* 使用SQLContext.createDataFrame生成DF

```scala
import org.apache.spark.sql.{SaveMode, SQLContext, Row}
import org.apache.spark.sql.types.{StringType, StructField, StructType}

val schemaString = "userId gender age occupation zipcode"
val schema = StructType(schemaString.split(" ").map(filedName =>
                                                   StructField(fieldName, StringType, true)))
val userRDD2 = usersRdd.map(_.split(","))
  .map(p => Row(p(0), p(1).trim, p(2).trim, p(3).trim, p(4).trim))
  
val userDataFrame2 = sqlContext.createDataFrame(userRDD2, schema)  
```

读取csv，需要添加Maven依赖`com.databricks:spark-csv_2.10:1.2.0`

```scala
val userCsvDF = sqlContext.read
  .format("com.databricks.spark.csv")
  .load("/tmp/user.csv")
  .toDF("userID", "age")
```

Transformation: groupBy，通过scala的map可以取代特定的count函数，但是countDistinct不行

```scala
userDF.groupBy("age").agg(count('gender'), countDistinct('occupation')).show()
  
userDF.groupBy("age").agg("gender" -> "count", "occupation" -> "count").show()
```

### Spark SQL调优

* DataFrame缓存：`sqlContext.cacheTable("tableName")`或者`dataFrame.cache()`
* 参数调优
  * Reduce task数目：spark.sql.shuffle.partitions（默认是200）
  * 读数据时每个Partition大小: spark.sql.files.maxPartitionBytes（默认是128MB）
  * 小文件合并读：spark.sql.files.openCostInBytes（默认是4194304）
  * 广播小表大小：spark.sql.autoBroadcastJoinThreshold（默认是10485760），join时会广播小表，类似Hive的小表放左边