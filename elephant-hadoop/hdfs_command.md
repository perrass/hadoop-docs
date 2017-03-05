负载均衡器
限定份额

在hadoop中，用hadoop jar来运行，而不是用java运行，因为hadoop会自动导入环境变量而java不用

原始日至存储格式选择

* 文本文件
    + 不便于压缩
    + 不建议直接存成文本格式
* SequenceFile (flume导出)
    + 二进制格式，便于压缩，压缩格式作为元信息存入文件中
    + 以key - value存储每条日志

小文件优化

* 合并成大文件
    + mapReduce
    + Hadoop archive
* 保存到key/value系统中
    + HBase

压缩方法

* gzip
* lzo (解压快，用于常被访问的文件，mapReduce/Hive/Spark)
* snappy (同上)
* bzip2 (压缩率高，但解压慢，用于不长被访问的文件)

数据存储格式选择

* 原始日志存储格式选择SequenceFile
* 原使用户信息和商品信息采用列式存储格式(ORC或者Parquet)，有利于Hive查询
