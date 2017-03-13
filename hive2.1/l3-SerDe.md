## Hive序列化器与反序列化器

Hive使用SerDe(Serializer, Deserializer)接口去完成IO操作，接口的三个主要功能

1. 序列化：从Hive写FS
2. 反序列化：从FS写入Hive
3. 解释读写字段，文件到字段结构的桥梁

Hive核心组件：

1. Query Processor: 查询处理工具，源码ql包
2. SerDe: 序列化反序列化器，源码serde包
3. MetaStore: 元数据存储及服务，源码metastore (支持整个Hadoop的元数据服务，包括Impala, HBase)

序列化与反序列化的目的

1. 序列化将Hive格式输出为特定格式，包括分隔符(tab，逗号，CTRL-A)，Thrift协议
2. 反序列化将HDFS格式读入Hive内存中，包括：Java Integer/String/ArrayList/HashMap，Hadoop Writable类, 用户自定义类

AbstractSerde抽象类

* serialize() 接口 Object -> Writable
* deserialize() 接口 Writable -> Object

ObjectInspector: 用于解耦数据使用和数据格式，Hadoop Writable <-> SerDe <-> ObjectInspector <-> Hive Objects / Operators

* 在输入和输出端切换不同的输入/输出格式
* 在不同的Operator上使用不同的数据格式

Deserializer工作

```java
Object deserialized = deserializer.deserializer(value);  // value is a Writable
Object row = partTblObjectInspectorConverter.convert(deserialized);
```

