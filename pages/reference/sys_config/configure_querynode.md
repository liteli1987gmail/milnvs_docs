# 查询节点相关配置

本文介绍 Milvus 查询节点的相关配置。

查询节点在增量数据和历史数据上执行向量和标量数据的混合搜索。

在本节中，您可以配置查询节点端口、优雅时间等。

## `queryNode.gracefulTime`

| 描述                                                         | 默认值 |
| ------------------------------------------------------------ | ------ |
| <li>新插入数据可以被搜索的最小时间。</li><li>单位：毫秒</li><li>当搜索消息的时间戳早于查询节点系统时间时，Milvus 会直接执行搜索请求。</li><li>当搜索消息的时间戳晚于查询节点系统时间时，Milvus 会等待，直到查询节点系统时间和时间戳之间的时间差小于此参数，然后执行搜索请求。</li> | 0      |

## `queryNode.port`

| 描述             | 默认值 |
| ---------------- | ------ |
| 查询节点的 TCP 端口 | 21123  |

## `queryNode.grpc.serverMaxRecvSize`

| 描述                                                         | 默认值    |
| ------------------------------------------------------------ | --------- |
| <li>查询节点可以接收的每个 RPC 请求的最大大小。</li><li>单位：字节</li> | 2147483647 |

## `queryNode.grpc.serverMaxSendSize`

| 描述                                                         | 默认值    |
| ------------------------------------------------------------ | --------- |
| <li>查询节点在接收 RPC 请求时可以发送的每个响应的最大大小。</li><li>单位：字节</li> | 2147483647 |

## `queryNode.grpc.clientMaxRecvSize`

| 描述                                                         | 默认值    |
| ------------------------------------------------------------ | --------- |
| <li>查询节点在发送 RPC 请求时可以接收的每个响应的最大大小。</li><li>单位：字节</li> | 104857600 |

## `queryNode.grpc.clientMaxSendSize`

| 描述                                                         | 默认值    |
| ------------------------------------------------------------ | --------- |
| <li>查询节点可以发送的每个 RPC 请求的最大大小。</li><li>单位：字节</li> | 104857600 |

## `queryNode.stats.publishInterval`

| 描述                                                         | 默认值 |
| ------------------------------------------------------------ | ------ |
| <li>查询节点发布节点统计信息的间隔，包括段状态、CPU 使用率、内存使用率、健康状态等。</li><li>单位：毫秒</li> | 1000   |

## `queryNode.dataSync.flowGraph.maxQueueLength`

| 描述                                                         | 默认值 |
| ------------------------------------------------------------ | ------ |
| <li>查询节点中流图的任务队列缓存的最大大小。</li><li>单位：MB</li><li>查询节点使用流图来订阅和组织消息流。</li> | 1024   |

## `queryNode.segcore.chunkRows`

| 描述                                                         | 默认值 |
| ------------------------------------------------------------ | ------ |
| 由 Segcore 将段划分为块的行数。 | 1024   |

## `queryNode.segcore.InterimIndex`

| 描述                                                         | 默认值 |
| ------------------------------------------------------------ | ------ |
| 是否为增长中的段和尚未索引的封闭段创建临时索引，以提高搜索性能。<br/><ul><li>Milvus 最终会封闭并索引所有段，但启用此功能可以优化数据插入后立即查询的搜索性能。</li><li>这默认为 `true`，表示 Milvus 为增长中的段和在搜索时尚未索引的封闭段创建临时索引。</li></ul> | true    |