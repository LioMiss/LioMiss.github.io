---
layout: post
title: "Unity ECS系列（一）概述"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - ECS
  - Unity
---

ECS架构由身份（entities实体）、数据（components组件），与行为（systems系统）组成，这是一种面向数据的架构。System从Component获取数据进行处理，然后对entities进行索引。
下图说明了这三个部分是如何协作的：  
![Xgu4qf.png](https://s1.ax1x.com/2022/06/11/Xgu4qf.png)
在这张图中，System读取Translation和Rotation组件，将他们相乘然后更新相应的LocalToWorld组件。  
实体A和实体B拥有Render组件，而C没有，但此System并不会受到影响，因为他们并不关心Renderer组件。
你可以设置一个需要Renderer组件的系统，在这种情况下，这个系统将忽略C的组件。或者你也可以设置一个排除具有Renderer组件的系统，然后忽略A和B的组件。

### 原型
组件类型的唯一组合称为实体原型（EntityArchetype），举个例子：一个3D物体可能拥有用于其进行世界变换的组件，Translation用于线性运动、Rotation用于旋转、Renderer用于视觉呈现。3D对象的每个实例都对应一个实体，但由于它们共享同一组组件，ECS将其归类为同一个原型。  
[![Xgl3l9.png](https://s1.ax1x.com/2022/06/11/Xgl3l9.png)](https://imgtu.com/i/Xgl3l9)
在此图中，实体A和B共享原型M，而实体C具有原型N。如果需要更改实体的原型，可以在运行时添加或删除组件。例如：从实体B中删除Renderer组件，将导致它的原型变成N。

#### Editor中查看原型
在Editor中，[![XglgTf.png](https://s1.ax1x.com/2022/06/11/XglgTf.png)](https://imgtu.com/i/XglgTf)图标表示原型。可以在[Archetypes window](https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/editor-archetypes-window.html)中进行查看。

### 内存块
实体的原型决定了ECS在何处存储该实体的组件，ECS以块的形式分配内存，每个块由一个ArchetypeChunk对象表示。块始终包含单个原型的实体，当一个内存块满了，ECS会分配一个新的内存块出来给新实体使用。如果实体原型由于添加或删除组件而发生了变化，ECS会将该实体的组件移动到不同的区块。
[![XgGUud.png](https://s1.ax1x.com/2022/06/11/XgGUud.png)](https://imgtu.com/i/XgGUud)
这种体系结构提供了原型与块之之间的一对多关系。这意味着查找具有特定组件的实体只需要搜索现有的原型，而不用搜索所有实体。  
ECS不按特定的顺序在区块中存储实体。当一个实体创建或变成新原型时，ECS将其放入第一个块中，虽然还有空间剩余，但各个块之间仍然是紧密排布的；当实体从原型中移除时，ECS会将区块中最后一个实体的组件移动到组件数组新腾出的位置上。  
Note:原型中共享的值同样决定了实体存储在哪个块中，给定区块中的所有实体对于任何共享组件都具有相同的值。如果更改共享组件中任意字段的值，会导致实体移动到不同的块，就像更改该实体的原型一样。如有必要将分配一个新的块。  
使用共享组件将原型中的实体分组，以便更有效的将它们一起处理。例如：混合渲染器定义其RenderMesh组件来实现这一点。

### 实体查询
要确定系统应该处理哪些实体，请使用EntityQuery。实体查询在现有原型中搜索匹配的组件的原型，可指定以下条件进行查询：
All -原型必须包含所有指定的组件类型  
Any -原型包含至少一个指定的组件类型  
None -原型必须不包含指定的组件类型  
实体查询返回一个块的列表，然后可以使用[IJobEntityBatch](https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/ecs_ijobentitybatch.html)对这些块进行迭代。

### Jobs
要利用多线程，可以使用[JobSystem](https://docs.unity3d.com/2020.3/Documentation/Manual/JobSystem.html)。ECS提供SystemBase类以及Entities.ForEach以及IJobEntityBath Schedule()以及ScheduleParallel()函数，用于在主线程外转换数据。Entities.Foreach是使用最简单的，通常需要很少的代码就能实现。您可以将IJobChuck用于处理更复杂的情况。
ECS按照System的排列顺序在主线程上安排Jobs，在安排Jobs时，ECS会跟踪哪些Jobs会读写哪些组件，读取组件的Jobs依赖于之前写入同一组件的任何计划的Job，反之依然。Jobs调度器使用Job依赖关系来确定哪些Job可以并行运行，哪些Job必须按顺序运行。

### System结构
ECS先按世界再按组来组织系统。默认情况下，ECS创建一个默认的世界，其中包含预定义的组集。它会查找所有可用的System，实例化它们，并将它们添加到默认世界中的预定义组中。  
可以指定组内System的更新顺序。组也是一种System，因此可以将组添加到另一个组中，并像其它系统一样指定其顺序，组中的所有System将在下一个组或者System更新之前更新。如果不指定顺序，ECS将以不依赖于创建顺序的方式System插入更新顺序，换句话说，即使没有明确指定顺序，同一组系统在其组中也总是以相同的顺序更新。  
System更新发生在主线程上，但是System可以使用Jobs将工作卸载到其他线程。[SystemBase](https://docs.unity3d.com/Packages/com.unity.entities@0.50/api/Unity.Entities.SystemBase.html)提供了一种创建和调度Jobs的简单方法。  
有关System创建、更新顺序、以及可用于组织系统的属性的更多信息，请参阅[System Update Order](https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/system_update_order.html)

### ECS创作
在Unity中创建游戏或应用程序时，可以使用GameObjects和MonoMehabiours创建转换系统，将这些unity引擎对象映射到entities，有关详细信息，请参见[Creating Gameplay](https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/gp_overview.html)