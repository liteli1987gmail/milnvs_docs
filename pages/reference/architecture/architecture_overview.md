# Milvus 架构概览

Milvus 是一个快速、可靠、稳定的向量数据库，专门为相似性搜索和人工智能而构建。它基于流行的向量搜索库，包括 Faiss、Annoy、HNSW 等，旨在对包含数百万、数十亿甚至数万亿向量的密集向量数据集进行相似性搜索。在继续之前，请熟悉[嵌入检索的基本原理](glossary.md)。

Milvus 还支持数据分片、数据持久性、流式数据摄取、向量和标量数据的混合搜索以及许多其他高级功能。该平台按需提供性能，并且可以针对任何嵌入检索场景进行优化。我们建议使用 Kubernetes 部署 Milvus，以实现最佳的可用性和弹性。

Milvus 采用了共享存储架构，具有存储和计算分离以及计算节点的水平可扩展性。遵循数据平面和控制平面分离的原则，Milvus 由 [四层](four_layers.md) 组成：接入层、协调服务、工作节点和存储。这些层次在扩展或灾难恢复方面是相互独立的。

![架构图](..//milvus_architecture.png "Milvus 架构。")

## 接下来

- 了解更多关于 Milvus 中的 [计算/存储分离](four_layers.md)
- 了解 Milvus 中的 [主要组件](main_components.md)