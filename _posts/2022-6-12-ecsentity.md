---
layout: post
title: "Unity ECS系列（二）实体Entities"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - ECS
  - Unity
---

Entities（实体）是实体、组件、系统体系结构中的三个主要元素之一。它代表游戏或者应用程序中的单个“事物”。实体既没有行为也没有数据，它标识了哪些数据块属于一起。System提供行为，component存储数据。  
一个Entity本质上是一个ID，最简单的方法是将其视为一个轻量级的GameObject，默认情况下甚至没有名字。EntityID是固定的，你可以用它们来存储对其他组件或Entity的引用。例如，hierarchy中的子Entity可能需要引用其父Entity。  
EntityManager管理世界中所有的实体，维护实体列表，并组织与实体关联的数据以优化性能。  
尽管实体没有类型，但实体组可以按照与其关联的数据组件的类型进行分类。创建实体并向其中添加组件时，EntityManager会跟踪现有实体上组件的唯一组合，也就是前文所提到的原型（Archetype），向实体添加组件时，EntityManager会创建EntityArchetype结构。可以使用现有的EntityArchetype创建符合该原型的新实体，也可以提前创建EntityArchetype并用它来创建实体。

### 创建Entities
创建实体的最简单方法是在UnityEditor中，你可以将ECS设置为在运行时将放置在场景中的GameObject和Prefabs转换为Entities。对于游戏或应用程序中更具动态性的部分，可以创建一个生成系统，在Jobs中创建多个实体。最后，您可以使用EntityManager.CreateEntity函数，一次创建一个实体，但这通常是效率最低的方法。

#### 使用EntityManager创建Entities
使用EntityManager.CreateEntity函数创建实体，ECS与EntityManager在相同的世界里创建实体。  
可以通过以下方式逐个创建实体：
创建一个带有指定数组内组件的实体。  
创建一个带有指定原型内组件的实体。  
复制一个现有的实体，包括其现有数据。  
创建一个没有组件的实体。（可以随时对其添加组件）

也可以一次创建多个实体：  
使用CreateEntity用具有相同原型的新实体填充NativeArray。  
使用Instantiate对现有实体复制，填充NativeArray。

### 添加和移除组件
创建实体后，可以添加或移除组件，执行此操作时，受影响的实体原型会发生变更。EntityManager会将更改后的数据移动到新的内存块中，并将组件数组压缩到原始内存块中。  
无法在Jobs中对会导致结构更改的实体进行更改（即添加或删除组件会更改SharedComponentData中的值，并销毁实体），因为这些更改可能会使Job正在处理的数据无效。但是你可以通过EntityCommandBuffer进行这些更改，等到Job完成后再执行这个Command。  
EntityManager提供了从单个实体以及NativeArray中的所有实体中删除组件的功能。相关信息参阅文档[Components](https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/ecs_components.html)

### 迭代实体
迭代具有匹配的组件集的所有实体是ECS体系结构的核心。请参见[Accessing entity data](https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/chunk_iteration.html)

### Editor中的实体
在Editor中，![X22xN6.png](https://s1.ax1x.com/2022/06/13/X22xN6.png)图标表示实体。可以在[Entities windows and Inspectors](https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/editor-workflows.html)中进行查看

### 访问实体数据
在使用ECS系统时，迭代数据是最常见的任务之一，ECS系统通常处理一组实体，从一个或多个组件读取数据，执行计算，将结果写入另一个组件。迭代实体和组件最有效的方法是在可并行Job中按顺序处理组件，这有效利用了所有可用核心，同时数据存储的位置可有效提高CPU的缓存命中。  
ECS API提供了许多迭代的方法，每种方法都有自己的性能影响和局限，您可通过一下方式迭代ECS数据：  
SystemBase.Entities.ForEach -逐个实体处理组件数据的最简单有效的方法  
IJobEntity -迭代ECS组件数据的第二种最简单方法，当你想要写一次代码多次调用可以选择这种方式  
IJobEntityBatch -迭代包含匹配实体的内存块，Job Execute函数可以使用for循环类迭代每个块中的成批元素，对于比实体更复杂的情况，可以使用IJobEntityBatch.ForEach，同时也可以保证效率  
手动迭代 -如果前面的方法不能满足需求，可以手动迭代实体或块，例如：可以使用IJobParallelFor之类的Job来迭代包含要处理的实体或实体块的NativeArray  
[EntityQuery](https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/ecs_entity_query.html)类提供了一种构建数据视图的方法，该视图只包含给定算法或流程所需的特定数据。上面列表中的许多迭代方法都显式或内部使用EntityQuery。

重要提示：新代码中不应使用一下迭代类型：
IJobChunk  
IJobForEach  
IJobForEachWithEntity  
ComponentSystem  
这些类型正在逐步淘汰。

### 使用EntityQuery查询数据
要读取或写入数据，必须先找到要更改的数据。ECS中的数据存储在组件中，ECS根据所属实体的原型在内存中分组。您可以使用EntityQuery查询ECS数据，该数据只包含给定算法或流程所需的特定数据。  
您可以使用EntityQuery执行以下操作：  
运行Job以处理选定的实体和组件  
获取包含所有选定实体的NativeArray  
获取所选组件的NativeArray（按组件类型）  
EntityQuery返回的实体和组件数组保证是“并行的”，即相同的索引值始终应用于任何数组中的相同实体。  

### World
一个世界（World）将实体组织成孤立的组，一个世界拥有一个EntityManager和一组Systems，在一个世界中创建的实体只在该世界中生效，但可以通过EntityManager.MoveEntitiesFrom转移到其他世界。系统只能访问同一世界中的实体，您可以创建任意多个世界。  
默认情况下，Unity在应用程序启动时创建默认世界，unity实例化所有系统（继承自ComponentSystemBase）并将它们添加到此默认世界，Unity还可以在编辑器中创建专门的世界。比如：它为仅在编辑器中而不是在Playmode中运行的实体和系统创建了一个编辑器世界，还创建了用于管理游戏对象到实体的转换的转换世界。有关可以创建的不同类型世界的示例，请参见[WorldFlags](https://docs.unity3d.com/Packages/com.unity.entities@0.50/api/Unity.Entities.WorldFlags.html)  
使用[World.DefaultGameObjectInjectionWorld](https://docs.unity3d.com/Packages/com.unity.entities@0.50/api/Unity.Entities.World.DefaultGameObjectInjectionWorld.html#Unity_Entities_World_DefaultGameObjectInjectionWorld)访问默认世界。

#### 管理系统
世界对象提供了创建、访问和从世界中删除系统的方法。大多数情况下，可以使用GetOrCreateSystem来获取系统实例。

#### 时间
系统时间属性的值由系统所在的世界控制，默认情况下，Unity为每个世界创建一个TimeData实体，该实体由UpdateWorldTimeSystem实例更新，以反映上一帧以来经过的时间，系统的时间属性是当前世界时间的别名。  
FixedStepSimulationSystemGroup对时间的处理与其他系统组不同，以固定间隔更新，如果固定间隔是一帧中很小的一部分，则可能会在每帧更新多次。  
如果需要更好地控制世界中的时间，可以直接使用World.SetTime。你可以用PushTime来更改世界时间，也可以用PopTime来返回到上一个时间。  

#### 自定义初始化
要在启动时手动初始化游戏，可以实现ICustomBootstrap接口，Unity使用默认世界时会调用这个接口，通过这种方式可以修改或者完全替换系统创建和初始化的过程。  
你还可以通过定义一下全局符号完全禁用默认世界的创建：  
UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP_RUNTIME_WORLD,禁用默认运行时世界的生成  
UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP_EDITOR_WORLD，禁用默认编辑器世界的生成  
UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP，同时禁用以上两种世界的生成  
然后，您的代码负责创建任何需要的世界，以及实例化和更新系统，您可以使用Unity 可编程PlayerLoop修改普通的Unity播放器循环方式，以便在需要时更新系统。  