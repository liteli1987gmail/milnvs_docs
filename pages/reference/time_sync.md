# 时间同步

本文介绍了 Milvus 中的时间同步机制。

## 概述

Milvus 中的事件通常可以分为两类：

- 数据定义语言（DDL）事件：创建/删除集合，创建/删除分区等。

- 数据操纵语言（DML）事件：插入，搜索等。

无论 DDL 还是 DML 事件，都会被标记一个时间戳，以指示该事件发生的时间。

假设有两个用户在 Milvus 中按以下时间顺序发起一系列 DML 和 DDL 事件。

| 时间戳 | 用户 1                   | 用户 2                   |
|:-----:|:------------------------:|:------------------------:|
|   t0  | 创建名为 `C0` 的集合。 |                         |
|   t2  |                         | 在集合 `C0` 上进行搜索。 |
|   t5  | 向集合 `C0` 插入数据 `A1`。 |                         |
|   t7  |                         | 在集合 `C0` 上进行搜索。 |
|  t10  | 向集合 `C0` 插入数据 `A2`。 |                         |
|  t12  |                         | 在集合 `C0` 上进行搜索。 |
|  t15  | 从集合 `C0` 删除数据 `A1`。 |                         |
|  t17  |                         | 在集合 `C0` 上进行搜索。 |

理想情况下，用户 2 应该能够看到：

- 在 `t2` 时，集合 `C0` 是空的。

- 在 `t7` 时，数据 `A1` 存在。

- 在 `t12` 时，数据 `A1` 和 `A2` 都存在。

- 在 `t17` 时，只有数据 `A2` 存在（因为数据 `A1` 在此之前已从集合中删除）。

当只有一个节点时，这种理想场景很容易实现。然而，Milvus 是一个分布式向量数据库，为了确保不同节点上的所有 DML 和 DDL 操作都能保持顺序，Milvus 需要解决以下两个问题：

1. 如上例中，如果两个用户在不同的节点上，他们的时钟时间可能不同。例如，如果用户 2 比用户 1 晚 24 小时，那么用户 1 的所有操作在第二天之前对用户 2 都是不可见的。

2. 可能存在网络延迟。如果用户 2 在 `t17` 对集合 `C0` 进行搜索，Milvus 应该能够保证在 `t17` 之前的所有操作都已成功处理并完成。如果由于网络延迟，`t15` 处的删除操作被延迟，那么用户 2 在 `t17` 进行搜索时很可能仍然能看到本应已删除的数据 `A1`。

因此，Milvus 采用了时间同步系统（timetick）来解决这些问题。

## 时间戳预言器（TSO）

为了解决前一节提到的第一个问题，Milvus 与其他分布式系统一样，提供了一个时间戳预言器（TSO）服务。这意味着 Milvus 中的所有事件都必须从 TSO 分配时间戳，而不是从本地时钟。

TSO 服务由 Milvus 中的根协调器提供。客户端可以在单个时间戳分配请求中分配一个或多个时间戳。

TSO 时间戳是一个 `uint64` 值，由物理部分和逻辑部分组成。下图展示了时间戳的格式。

![TSO_Timestamp](/TSO_Timestamp.png "TSO 时间戳。")

如上图所示，前面的 46 位是物理部分，即以毫秒为单位的 UTC 时间。最后 18 位是逻辑部分。

## 时间同步系统（timetick）

本节以数据插入操作为例，解释 Milvus 中的时间同步机制。

当代理接收到来自 SDK 的数据插入请求时，它会根据主键的哈希值将插入消息划分为不同的消息流（`MsgStream`）。

每个插入消息（`InsertMsg`）在发送到 `MsgStream` 之前都会被分配一个时间戳。

<div class="alert note">
  <code>MsgStream</code> 是消息队列的包装器，默认情况下在 Milvus 2.0 中是 Pulsar。
</div>

![timesync_proxy_insert_msg](/timesync_proxy_insert_msg.png "从多个代理插入数据到 MsgStreams 的示例。")

一个通用原则是，在 `MsgStream` 中，来自同一代理的 `InsertMsgs` 的时间戳必须是递增的。然而，来自不同代理的 `InsertMsgs` 没有这样的规则。

下图是一个 `MsgStream` 中的 `InsertMsgs` 的示例。该代码片段包含五个 `InsertMsgs`，其中三个来自 `Proxy1`，其余