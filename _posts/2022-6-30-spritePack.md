---
layout: post
title: "Unity 实时生成图集"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - Sprite
  - Unity
---

最近在做动态图集相关的东西，恰好之前有看过[《小米超神》技术总监王啸予：重度MOBA的优化之路](https://blog.uwa4d.com/archives/Severe_MOBA.html)，里面提到了一种图集分块算法，这个算法的思想本质上类似于BSP，本文会简单介绍一下这个算法的大体思路，同时也会对Unity官方的图集分块算法进行分析。

### DaVikingCode的图集分块算法
Unity和插件提供了许多构建图集的好方法，他们直接用于UnityEditor或与外部软件配合使用，在许多情况下都是完美的，但没有一个能够在运行时生成图集。  
图集的目标是在一个大纹理中填充尽可能多的子纹理，所以首先要做的是实现一个打包算法，我找到了一个AS3版本的开源算法[https://villekoskela.org/2012/08/12/rectangle-packing/](https://villekoskela.org/2012/08/12/rectangle-packing/)，所以我们把它改成C#版本即可。  
[![jlozhn.png](https://s1.ax1x.com/2022/07/02/jlozhn.png)](https://imgtu.com/i/jlozhn)  
打包完成后，我们就可以在一个缓存系统上工作，以便将生成的图集写入并保存在磁盘上，以便将来加载。除非要修改，否则无需重新生成。

#### 矩形打包算法
矩形打包的思想是将较小的矩形尽可能紧密的放置在较大的矩形容器里，这在生成包含多个子纹理的大纹理时特别有用。我的实现在主矩形中使用了空闲矩形的概念。压缩矩形总是放置在某个空闲矩形的左上角，它们完全适合该矩形，为了获得非常接近最优的填充，选择填充矩形适合的最左空闲矩形的顶部进行放置。  
最初只有一个空闲矩形，就是整个矩形本身。将第一个矩形放进来后，移除空闲矩形栈中的第一个矩形，并生成两个新的空闲矩形（如果空间被占满，则不会生成新的矩形，如果高或宽被占满，则会生成一个矩形）。加入新矩形与这个流程是一样的，与新加入矩形相交的任何空闲矩形都会被切割成围绕这个矩形的新的较小的空闲矩形，然后需要删除由空闲矩形完全包裹着的空闲矩形。  
下图显示了空闲矩形的组合方式。
[![j3rgcd.png](https://s1.ax1x.com/2022/07/02/j3rgcd.png)](https://imgtu.com/i/j3rgcd)
左侧图像显示，放置第一个矩形后分割出了两个空闲矩形，放置第二个矩形会将空闲矩形2划分为两个新的空闲矩形，并将空闲矩形1划分为三个新的空闲矩形，但是因为其中有被完全包含在更大的空闲矩形中，因此被删除了。

算法源码：[RectanglePacking](https://github.com/villekoskelaorg/RectanglePacking)，作者：villekoskelaorg

C#版本：[UnityRuntimeSpriteSheetsGenerator](https://github.com/DaVikingCode/UnityRuntimeSpriteSheetsGenerator)

### Unity官方的图集分块算法
Unity的分块算法，先对所有图片按从大到小进行排序，然后遍历，遍历过程中会不断对interiorw和interiorh进行修改，这两个变量表示已被占用的空间，每次遍历会查看这个空间内是否还有合适的位置放入图片，如果没有的话就会在右侧扩展这个空间，同时interiorw = interiorw + texture.width，直到所有的图片都放置完。  
相比于上面的方法，Unity官方的分块算法每次放入图片都需要逐像素扫描，所以速度会慢一些。不会对于空间的利用还是比较高的。