# 存储/计算分离

遵循数据平面和控制平面分离的原则，Milvus由四个层组成，这些层在可扩展性和灾难恢复方面是相互独立的。

## 接入层

由一组无状态代理组成，接入层是系统的前端层，也是用户的终端点。它验证客户端请求并减少返回的结果：

- 代理本身是无状态的。它使用Nginx、Kubernetes Ingress、NodePort和LVS等负载平衡组件提供统一的服务地址。
- 由于Milvus采用了大规模并行处理（MPP）架构，代理在将最终结果返回给客户端之前，会聚合并后处理中间结果。

## 协调服务

协调服务将任务分配给工作节点，并充当系统的大脑。它承担的任务包括集群拓扑管理、负载平衡、时间戳生成、数据声明和数据管理。

有三种协调器类型：根协调器（root coord）、数据协调器（data coord）和查询协调器（query coord）。

### 根协调器（root coord）

根协调器处理数据定义语言（DDL）和数据控制语言（DCL）请求，如创建或删除集合、分区或索引，以及管理TSO（时间戳Oracle）和时间滴答发行。

### 查询协调器（query coord）

查询协调器管理查询节点的拓扑和负载平衡，并从增长段移交到封闭段。

### 数据协调器（data coord）

数据协调器管理数据节点的拓扑，维护元数据，并触发刷新、压缩和其他后台数据操作。

## 工作节点

手臂和腿。工作节点是愚蠢的执行者，遵循协调服务的指令，并执行代理的数据操作语言（DML）命令。由于存储和计算的分离，工作节点是无状态的，并且可以在Kubernetes上部署以促进系统扩展和灾难恢复。有三种类型的工作节点：

### 查询节点

查询节点通过订阅日志代理检索增量日志数据，并将它们变成增长段，从对象存储中加载历史数据，并在向量和标量数据之间运行混合搜索。

### 数据节点

数据节点通过订阅日志代理检索增量日志数据，处理变异请求，并将日志数据打包成日志快照并存储在对象存储中。

### 索引节点

索引节点构建索引。索引节点不需要驻留在内存中，并且可以使用无服务器框架实现。

## 存储

存储是系统的骨干，负责数据持久性。它包括元存储、日志代理和对象存储。

### 元存储

元存储存储集合模式、节点状态和消息消费检查点等元数据的快照。存储元数据需要极高的可用性、强一致性和事务支持，因此Milvus选择了etcd作为元存储。Milvus还使用etcd进行服务注册和健康检查。

### 对象存储

对象存储存储日志的快照文件、标量和向量数据的索引文件以及中间查询结果。Milvus使用MinIO作为对象存储，并且可以轻松部署在AWS S3和Azure Blob上，这是世界上最受欢迎的两种成本效益高的存储服务。然而，对象存储具有高访问延迟，并按查询次数收费。为了提高其性能并降低成本，Milvus计划在基于内存或SSD的缓存池中实现冷热数据分离。

### 日志代理

日志代理是一个支持回放的pub-sub系统。它负责流式数据持久性、执行可靠的异步查询、事件通知和返回查询结果。它还确保工作节点从系统故障中恢复时增量数据的完整性。Milvus集群使用Pulsar作为日志代理；Milvus独立使用RocksDB作为日志代理。此外，日志代理可以轻松替换为Kafka和Pravega等流式数据存储平台。

Milvus围绕日志代理构建，并遵循“日志即数据”的原则，因此Milvus不维护物理表，而是通过日志持久性和快照日志保证数据可靠性。

![Log_mechanism](..//log_mechanism.png "日志机制。")

日志代理是Milvus的骨干。由于其固有的pub-sub机制，它负责数据持久性和读写分离。上面的图示显示了机制的简化描述，其中系统分为两个角色，日志代理（用于维护日志序列）和日志订阅者。前者记录所有更改集合状态的操作；后者订阅日志序列以更新本地数据并以只读副本的形式提供服务。pub-sub机制还为系统在变更数据捕获（CDC）和全球分布式部署方面的可扩展性留出了空间。

## 接下来

- 阅读[主要组件](main_components.md)以获取有关Milvus架构的更多详细信息。