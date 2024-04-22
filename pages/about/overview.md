---

id: overview.md
title: 什么是 Milvus
related_key: Milvus 概览
summary: Milvus 是一个开源向量数据库，专为 AI 应用开发、嵌入向量相似性搜索和 MLOps 设计。

---

# 引言

本页面旨在回答几个问题，为您提供 Milvus 的概览。阅读本页面后，您将了解 Milvus 是什么、它的工作原理，以及关键概念、为什么使用 Milvus、支持的索引和度量、示例应用、架构和相关工具。

## Milvus 向量数据库是什么？

Milvus 创建于 2019 年，其唯一目标是存储、索引和管理由深度神经网络和其他机器学习（ML）模型生成的大量[嵌入向量](#嵌入向量)。

作为一个专门设计用于处理输入向量查询的数据库，它能够在万亿规模上对向量进行索引。与主要处理遵循预定义模式的结构化数据的现有关系数据库不同，Milvus 从底层设计上就用于处理从[非结构化数据](#非结构化数据)转换而来的嵌入向量。

随着互联网的增长和演变，非结构化数据变得越来越普遍，包括电子邮件、论文、IoT 传感器数据、Facebook 照片、蛋白质结构等。为了让计算机理解和处理非结构化数据，这些数据使用嵌入技术转换为向量。Milvus 存储和索引这些向量。Milvus 能够通过计算它们的相似性距离来分析两个向量之间的相关性。如果两个嵌入向量非常相似，这意味着原始数据源也是相似的。

![工作流程](/public/assets/milvus_workflow.jpeg "Milvus 工作流程。")

## 关键概念

如果您是向量数据库和相似性搜索领域的新手，请阅读以下关键概念的解释，以获得更好的理解。

了解更多关于 [Milvus 术语表](glossary.md)。

### 非结构化数据

非结构化数据包括图像、视频、音频和自然语言，是一种不遵循预定义模型或组织方式的信息。这种数据类型约占世界数据的 80%，可以使用各种人工智能（AI）和机器学习（ML）模型转换为向量。

### 嵌入向量

嵌入向量是对非结构化数据（如电子邮件、IoT 传感器数据、Instagram 照片、蛋白质结构等）的特征抽象。从数学上讲，嵌入向量是一个浮点数或二进制数组。现代嵌入技术用于将非结构化数据转换为嵌入向量。

### 向量相似性搜索

向量相似性搜索是将向量与数据库进行比较以查找与查询向量最相似的向量的过程。近似最近邻（ANN）搜索算法用于加速搜索过程。如果两个嵌入向量非常相似，这意味着原始数据源也是相似的。

## 为什么选择 Milvus？

- 在大量数据集上进行向量搜索时具有高性能。
- 开发者优先的社区，提供多语言支持和工具链。
- 即使在中断事件中也具有云可扩展性和高可靠性。
- 通过将标量过滤与向量相似性搜索配对实现混合搜索。

## 支持哪些索引和度量？

索引是数据的组织单位。在搜索或查询插入的实体之前，您必须声明索引类型和相似性度量。**如果您不指定索引类型，Milvus 将默认执行暴力搜索。**

### 索引类型

Milvus 支持的大多数向量索引类型使用近似最近邻搜索（ANNS），包括：

- **FLAT**：FLAT 最适合于寻求在小型、百万规模数据集上获得完全精确和精确搜索结果的场景。
- **IVF_FLAT**：IVF_FLAT 是一种基于量化的索引，最适合于寻求在准确性和查询速度之间实现理想平衡的场景。还有一个 GPU 版本 **GPU_IVF_FLAT**。
- **IVF_SQ8**：IVF_SQ8 是一种基于量化的索引，最适合于寻求在磁盘、CPU 和 GPU 内存消耗非常有限的情况下显著降低资源消耗的场景。
- **IVF_PQ**：IVF_PQ 是一种基于量化的索引，最适合于寻求即使以牺牲准确性为代价也要实现高查询速度的场景。还有一个 GPU 版本 **GPU_IVF_PQ**。
- **HNSW**：HNSW 是一种基于图的索引，最适合于对搜索效率有高需求的场景。

有关更多详细信息，请参见 [向量索引](index.md)。

### 相似性度量

在 Milvus 中，相似性度量用于测量向量之间的相似性。选择一个好的距离度量可以显著提高分类和聚类性能。根据输入数据的形式，选择特定的相似性度量以获得最佳性能