## Yarn

产生背景

* 一个框架一个集群（MySQL, Hadoop, ELK), 共享模式统一管理多个框架
* 多种框架共享数据和硬件资源，减少数据移动带来的成本

Yet Another Resource Negotiator

* 集群资源管理系统
* 负责集群的统一管理和调度

基本架构 (Master - Slave)

* Resource Manager
  * 处理客户端请求
  * 启动监控ApplicationMaster
  * 监控NodeManager
  * 资源分配与调度
* Node Manager (和Data Node部署在一起，内存/CPU + 硬盘)
  * 单节点的资源管理和任务管理
  * 处理来自ResourceManager的命令
  * 处理来自ApplicationMaster的命令
* ApplicationMaster (每个应用程序一个，负责应用程序的管理和任务)
  * 数据切分
  * 为应用程序申请资源，进一步分配给内部任务
  * 任务监控与容错
* Container (对因任务运行环境的抽象, 资源隔离)
  * 描述任务运行资源
  * 描述任务启动命令
  * 描述任务运行环境 

运行过程

1. Client (hadoop jar xxx) -> Resource Manager (客户端会处理，每个Map多少资源，Reduce多少资源，ApplicationMaster是什么，需要多少CPU多少内存)
2. Resource Manager -> Node Manager 启动对应的Application Master
3. Node Manager -> Application Master 汇总一共需要多少CPU，多少内存 (有多少Map tasks, 多少Reduce tasks)
4. Application -> Resource Manager Application Master 向Resource Manager申请资源, 并收到有多少资源现在可以使用, 在哪个节点上 (以容器形式).
5. Application Master -> Node Manager 找到对应的Node Manager
6. Node Manager -> Tasks 启动对应的Tasks

容错

* Resource Manager 基于ZooKeeper实现HA
* Node Manager RM将失败任务告诉对应的AM, AM决定如何处理失败的任务 (可以视而不见, 这1G我不要了!!!)
* Application Master 失败后RM负责重启, AM需处理内部任务的容错问题,  AppMaster 会保存已运行的Tasks

调度

* 双层调度: RM -> AM, AM -> Tasks
* 资源不够时, 会为Task预留, 直到资源充足 (与Mesos的all or nothing策略不同), 但如果资源过高, 会导致资源利用率降低
* 多种资源调度器
  * FIFO
  * Fair Scheduler
  * Capacity Scheduler
* 多租户资源调度器
  * 支持资源按比例分配
  * 支持层级队列划分方式
  * 支持资源抢占

资源隔离

* 资源隔离: 运行着不同任务的"Container" 提供可独立使用的计算资源, 以避免它们之间相互干扰
* 内存隔离
  * 基于线程监控的方案 
  * 基于Cgroups的方案
* CPU隔离
  * 默认不进行隔离
  * 基于Cgroups的方案

---

[Hadoop Yarn 资源隔离实现原理](http://blog.csdn.net/a860mhz/article/details/50618555)

> 对CPU而言, 使用量的大小不会直接影响到应用程序的死亡, 所以采用Cgroup. 对内存而言, 使用量大小决定应用程序的死亡, Cgroup会严格限制应用程序的内存使用上限, 如果使用Cgroup则程序会被杀死, 因此使用线程监控的方式.
>
> **为什么应用程序的内存会超过预先设定的上限值?**
>
> 这里的应用程序特质Yarn Container, 他是Yarn NodeManager 通过创建子进程的方式启动的, Java创建子进程时采用"fork() + exec()"的方案, 子进程启动瞬间, 它的内存使用量与父进程是一致的, 然后子进程恢复正常. 也就是说, Container的创建过程中可能会出现内存使用量超过预先设定的上限值的情况 (取决于NodeManager的内存使用量). 此时, 如果使用Cgroup进行内存资源隔离, 这个Container可能会被kill

---



