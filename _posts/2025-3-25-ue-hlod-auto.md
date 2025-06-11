---
layout: post
title: "UE5世界分区HLOD"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - Unreal Engine
  - Unreal
  - World Partition
  - HLOD
---

## HLOD

### 定义

HLOD是一组Actor的视觉显示，旨在玩家离目标Actor距离较远时，替换这些Actor，它通常是一个基于原Actor几何体构建的单一网格和材质，但进行了简化，通过减少DrawCall、顶点数、降低精度来减少开销，提高性能。

世界分区HLOD与传统的HLOD不同之处在于，它们不与级别相关联，它们是根据世界分区网格生成的，无需手动管理Actor集群。

在WorldSettings中设置HLOD Layer：

Open image-20250117-083224.png
image-20250117-083224.png  
其逻辑与分区Grid设置类似，勾选Is Spatially Loaded之后会按照Loading Range来设置流送。如果不勾选则默认激活。

### HLOD层
世界分区HLOD由HLOD层资产控制，可以同时使用多个HLOD层。  
![pEHnuCQ.png](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/bede73ee-45a3-4a64-913c-ed1accff2f87/image_1.png  )

新建HLOD层资产的步骤如下：  
打开 内容侧滑菜单（Content Drawer） ，点击 + 添加（+ Add） 打开菜单。找到 杂项（Miscellaneous） 菜单，然后选择 HLOD层（HLOD Layer） 资产。（右键单击 内容侧滑菜单（Content Drawer） ，打开同一个菜单。）  
双击新HLOD层，打开资产编辑器窗口。  
![pEHnuCQ.png](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/af73adf4-f089-4696-b0fc-d7134dd912c9/image_2.png)

#### HLOD层类型
UE目前有五种HLOD层类型：
![pEHyV4U.md.png](https://s21.ax1x.com/2025/04/30/pEHyV4U.md.png)
|层类型|说明|
|---|---|
|实例化（Instancing）|	为那些资产使用最低细节级别 (LOD) 设置将此类层中的静态网格体资产替换为实例化静态网格体 (ISM) 组件。此类型是理想的替代物网格体，如树木和枝叶。|
|合并网格（Merged Mesh）|	此层类型中的静态网格体将合并，以生成单个代理网格体。|
|简化网格（Simplified Mesh）|	此层类型中的静态网格体将合并，以生成单个代理网格体，然后执行网格体简化。|
|近似网格（Approximated Mesh）|此曾类型中的StaticMesh将合并，生成单个Mesh，并执行Mesh简化，与Simplified Mesh不同的地方是会使建筑物空心化|
|自定义（Custom）|自定义层类型，可以自定义HLOD层的行为|

#### HLOD层参数设置
设置HLOD层的参数需要与美术、ta等沟通，确定合适的参数，如：使用哪些贴图，是否开启Nanite等。  

#### HLOD层划分
一个世界可以有多个HLOD层，用于不同类型的资产，如简化网格的建筑和实例化的植被，它们可以设置为不同的HLOD层，单独设置其单元格大小和加载范围。

### HLOD自动Build
由于Build HLOD需要消耗大量时间，因此需要部署自动化流水线去自动Build HLOD，并提交结果，具体的Build脚本可以用Build Graph实现，UE引擎中提供了一个示例： 
```
Engine\Build\Graph\Examples\BuildWorldPartitionHLODs.xml
```
可以用如下命令执行：
```
.\RunUAT.bat BuildGraph -Script="Engine\Build\Graph\Examples\BuildWorldPartitionHLODs.xml" -Target="HLOD Generation" -set:MapName=/Game/YourMapName
```

参考资料：  
[官方文档](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/world-partition---hierarchical-level-of-detail-in-unreal-engine?application_version=5.5#%E5%88%9B%E5%BB%BAhlod%E5%B1%82)  
