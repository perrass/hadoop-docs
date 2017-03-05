HDFS基本架构与原理
===

问题

1. HDFS多副本放置策略，默认的3个副本如何选三个节点存放
    + 同Client的节点上
    + 不同机架的节点上(同机架的错误率比不同的高, 需要在配置文件来配置不同机架)
    + 与第二个副本同一机架的另一个节点上(不再选机架是因为中间交换机的带宽是瓶颈)
    + 修改副本数
        + hadoop dfs -setrep -w 4 /path
        + hadoop dfs -setrep -R -w 4 /dir
2. 1G文件如何写进HDFS
3. 为什么HDFS不适合储存小文件
    + 元信息存储在NameNode内存中，一个节点的内存是有限的
    + 存取大量小文件消耗大量的寻道时间
    + 一个block元信息消耗150 byte内存

### HDFS概述

* Hadoop Distribute File System
* master-slave设计, NameNode-DataNode
    + HDFS-client
        + 文件切分
        + 与NameNode交互，获取文件位置信息
        + 与DataNode交互，读取或写入数据
        + 管理HDFS
        + 访问HDFS
    + Active NameNode
        + 管理HDFS的名称空间
        + 管理数据块映射信息
        + 配置副本策略
        + 处理客户端读写请求
    + StandBy NameNode
        + NameNode的热备
        + 定期合并fsimage和fsedits推送给NameNode
        + 当Active NameNode出现故障时, 更新为Active NameNode
    + DataNode
        + 储存实际的数据块
        + 执行数据块读取
* Zookeeper
    + 协调Active NameNode和StandBy NameNode
* fsimage, edits
    + fsimage: 元数据镜像文件，保存文件系统的目录树
    + edits: 元数据操作日志，针对目录树的修改操作
    + 内存中的镜像 = fsimage + edits
    + edits文件过大将导致NameNode重启速度慢
    + StandBy NameNode定期合并

---

优点

* 高容错
    + 数据自动保存多个副本
    + 副本丢失后，自动恢复
* 批处理
    + 移动计算而非数据
    + 数据位置暴露给计算框架
* 适合大数据处理
    + 数据量级
    + 文件数量
    + 节点规模
* 流式文件访问
    + 一次性写入，多次读写
    + 保证数据一致性

缺点

* 低延迟数据访问
    + 毫秒级
    + 低延迟(latency)和高吞吐率(throughput)的折中，牺牲低延迟
* 小文件存储
    + 占用NameNode大量内存
    + 寻道时间(磁头从开始移动到移动到数据所在磁道所需要的平均时间，或计算机发出一个寻指命令，到对应目标数据被找到所需的时间)超过读取时间 (1G单文件和1G100万文件的拷贝)
* 并发写入、文件随机修改
    + 一个文件只能有一个写者 (**多个线程写入仅会有一个成功，其他线程抛异常**)
    + 仅支持append

---

场景

４个文件(0.5T, 1.2T, 50G, 100G)，如果将这四个文件写到7台(10*1T)的集群里

将文件分为不同的块(block, 一般是128MB)，将不同的块拷贝的不同的机器上，同时维护一张表

---

> 为什么是128MB?

数据传输时间超过寻道时间(提高吞吐率)

---

name|other
----|-----
file3|block1, block2, block3...
block1|node1, node2, node3...
block2|node2, node3, node4...

* 文件的负载均衡
* 被计算框架并行处理, 还是受节点数和带宽影响

---

读写机制

* 写，客户端随机选取一个DataNode传输一个小的packet(e.g.64KB), 这个DataNode收到后将这个packet传到下一个packet，同时收到新的小packet，这并非是一个并行的流程，因为如果是客户端到DataNode的并行，当DataNode节点多了，会受到客户端带宽限制，而此时贷款会平摊，此外，**两者性能差不多**
* 读, NameNode对block1分配一个DataNode, 客户端读取，然后NameNode对block2分配一个DataNode，客户端再读取

---

可靠性策略

* 文件完整性
    + CRC32校验
    + 用其他副本取代损坏文件
* Heartbeat
    + DataNode定期向NameNode发送Heartbeat
* 元数据信息
    + FSImage(文件系统镜像), Editlog(操作日志)
    + 多份存储
    + 主备NameNode实时切换
