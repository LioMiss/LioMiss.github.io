---
layout: post
title: "UE5世界分区性能监测"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - Unreal Engine
  - Unreal
  - World Partition
---

## 前言
之前介绍了世界分区的基本概念，以及使用方法，但在实际使用过程中，我们需要关注世界分区的性能开销，本文将介绍如何监测世界分区的性能。
## 流水线监测
地图运行之前，UE会生成一个StreamingGeneration的log文件，记录了地图的网格划分信息，包括网格单元的数量，网格单元的详细信息等，我们可以通过这个log文件来初步了解地图的网格划分情况，从而评估世界分区的性能开销。
### 值得关注的数据
#### Always loaded Actor Count: 214
始终加载的Actor数量，及Actor名字
#### Content of Cell XXXX_MainPartition_L0_X-3_Y-1_Z0
名字中的L0表示Cell的分级，除了明面上的Grid分层，UE实际上对Grid中的Cell还会再分级，根据Actor包围盒的大小，会将Cell划分为不同大小的Cell，L0表示最底层的Cell，L1表示下一级Cell，以此类推，L0的Cell最小，L1次之，以此类推，正常来说，应该尽可能减少级别，因为当一个Cell过于大时，它基本上是始终加载的，也就失去了流送的意义和价值，不如直接将里面的Actor设置为始终加载，从而减少网格遍历本身的开销。

## 运行时监测
运行时，可以输入控制台指令：wp.Runtime.ToggleDrawRuntimeCellsDetails，查看每个网格单元加载的耗时信息。

参考资料：  
[世界分区官方文档](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/world-partition-in-unreal-engine?application_version=5.5)  
[世界分区社区文档](https://dev.epicgames.com/community/learning/knowledge-base/r6wl/unreal-engine-world-building-guide)

