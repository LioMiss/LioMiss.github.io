---
layout: post
title: "《Shader入门精要》回顾"
subtitle: '温故而知新'
author: "LioMiss"
header-style: text
tags:
  - Shader
  - Unity
---

作为图形学的入门书籍，特别是作为Unity shader的入门书籍，这本书可以说得上是经典，毕竟unity shader相关的中文资料少之又少，而且，书中对于Unity shader的应用讲得很全面，详细。写这篇文章的目的，一为巩固，二为扩展，希望能读出一点新东西。


### 全书知识结构
![Shader入门](https://s2.ax1x.com/2019/10/25/K0PCB4.md.png "全书知识结构"){:height="600" width="800"}
### 渲染流水线
渲染，本质上来说计算机经过一系列计算和操作，将数据转化为图像并在屏幕中显示出来。而渲染流水线，就是中间经过的那一些列计算和操作。而经过多年的发展，现在的渲染流水线主要分为三个阶段，即：应用阶段、几何阶段、光栅化阶段。
[![K0mFET.png](https://s2.ax1x.com/2019/10/26/K0mFET.png)](https://imgchr.com/i/K0mFET)
#### 应用阶段
说白了就是CPU在这一阶段，将渲染所需要的各种数据，准备好，并通过Draw Call命令GPU进行下一阶段的渲染。
#### 几何阶段
[![K0mkUU.md.png](https://s2.ax1x.com/2019/10/26/K0mkUU.md.png)](https://imgchr.com/i/K0mkUU)
几何阶段又有几个子阶段，其中最为重要的就是顶点着色阶段，也是Unity Shader中常用的顶点着色器所作用的阶段，在顶点着色阶段，一个最基本的工作是进行坐标变换，将传过来的坐标数据从模型空间转换到齐次裁剪空间。其次就是进行逐顶点光照。  
另外一个重要的子阶段是裁剪，裁剪的作用是把那些在摄像机视野范围外的物体裁掉。  
最后一个阶段，屏幕映射则是把每个图元的坐标转换到屏幕坐标系。 
#### 光栅化阶段
光栅化阶段中最常用的是像素着色阶段，在Unity Shader中以片元着色器的形式使用，片元着色器的作用是根据顶点数据的插值，计算并输出这些点的颜色值。  
最后一个阶段合并阶段，虽然不可编程，但是高度可配置，我们可以通过配置一些参数，来处理可见性问题和和实现一些透明效果。
### 数学基础
计算机图形学之所以深奥难懂，很大程度上是因为它是建立在虚拟世界上的数学模型。而图形学相关的数学模型，无非是点、线、面、体，而它们其实都离不开坐标系。  
#### 笛卡尔坐标系
坐标系可以说是再熟悉不过了，而计算机图形学的坐标系和我们曾经学过的坐标系唯一的区别就是在计算机中会使用不同的坐标系，OpenGL和DirectX使用的就是不同方向的坐标系。
![K0nWSH.png](https://s2.ax1x.com/2019/10/26/K0nWSH.png)  
扩展到三维坐标系之后又有左手坐标系和右手坐标系之分。  
[![K0nvXn.png](https://s2.ax1x.com/2019/10/26/K0nvXn.png)](https://imgchr.com/i/K0nvXn)  
[![K0nj6s.png](https://s2.ax1x.com/2019/10/26/K0nj6s.png)](https://imgchr.com/i/K0nj6s)  
Unity在模型空间中使用的是左手坐标系，而在观察空间中，使用的是右手坐标系。
#### 点和矢量
主要涉及到一些计算公式和概念，如矢量的模，矢量和标量的乘法、除法，矢量的加法和减法，单位矢量，点积，叉积。
#### 矩阵
矩阵在三维数学中扮演着举足轻重的地位，但是它的表示形式相对复杂，计算也复杂得多，一些基本的运算，如矩阵和标量的乘法、矩阵和矩阵的乘法，一些特殊的矩阵，如：单位矩阵、方块矩阵、转置矩阵、逆矩阵、正交矩阵。  
而矩阵的几何意义则是难点和重点，矩阵可以表示平移、缩放、旋转、错切、镜像、投影等等变换。   
同样，矩阵也可以用来转换坐标空间，因为坐标空间的变换本质上就是对坐标系进行平移和旋转甚至缩放。  
### 光照
模拟真实世界的光照需要考虑物理现象：  
光线从光源中被发射出来。 
光线的反射和散射。  
这些被反射的光线如何被眼睛所接收到。  
#### 模拟光源
在Unity中，一共有四种光源，平行光、点光源、聚光灯、面光源。  
平行光可以照亮的范围是没有限制的，只有方向和强度的不同。而点光源是一个点发出的，向所有方向延申的光，可以调节他的范围和位置。聚光灯可以照亮的区域是一块锥形区域，聚光灯可以表示一个由特定位置出发，向特定方向延申的光。  
在实时渲染中，通常会指定多个Pass去渲染不同的光，如BasePass处理场景中的环境光，AdditionalPass处理其他的平行光、点光源、聚光灯等。
#### 反射和散射
标准的光照模型中，进入摄像机的光线分为四个部分，分别是：自发光、高光反射、漫反射、环境光。其中自发光是指直接由光源发射进入摄像机，而不经过任何物体的反射，这种光一般不会照亮周围物体的表面，但是在Unity5之后的版本中，因为加入了新的全局光照系统，则可以模拟这类光对周围物体的影响了。  
漫反射光照用于对那些被物体表面随机散射到各个方向的辐射度进行建模的。在漫反射中，视角的位置是不重要的，因为反射是完全随机的。漫反射光照符合兰伯特定律，即：反射光线的强度与表面法线和光源方向之间的夹角 的余弦值成正比。它的计算公式为：
$$
c_{diffuse} = (c_{light}\cdot m_{diffuse})max(0, \hat n \cdot \hat l)
$$
其中$\hat n$是表面法线，$\hat l$是指向光源的单位矢量，$m_{diffuse}$是材质的漫反射颜色，$c_{light}$是光源颜色。  
高光反射是一种经验模型，它不符合真实世界中的高光反射现象。它可用于计算那些沿着完全镜面反射方向被反射的光线，让物体看起来是有光泽的，如金属材质。  
高光反射的计算公式为：
$$
c_{specular}=(c_{light} \cdot m_{specular}) max(0, \hat v \cdot \hat r)^{m_{gloss}}
$$
其中：
$$
\hat r = 2(\hat n \cdot \hat l)\hat n - \hat l
$$
其中,$m_{gloss}$是材质的光泽度，也称为反光度，$c_{light}$则是光源的颜色和强度。
![Kr4154.png](https://s2.ax1x.com/2019/10/27/Kr4154.png)
#### 光照衰减
使用一张纹理作为查找表来在片元着色器中计算逐像素光照的衰减。
#### 阴影
为了使场景看起来更加真实，具有深度信息，通常希望光源可以把一些物体的阴影投射到其他物体上。  
Unity中使用了屏幕空间的阴影映射技术，Unity会首先调用LightMode为ShadowCaster的Pass来得到可投射阴影的光源的阴影映射纹理以及摄像机的深度纹理，然后根据光源的阴影映射纹理和摄像机的深度纹理来得到屏幕空间的阴影图。  
### 纹理
纹理的作用就是用一张图片来控制模型的外观，即纹理映射技术，主要是对应像素的颜色。
#### 简单纹理
![Kr5YFg.png](https://s2.ax1x.com/2019/10/27/Kr5YFg.png)  
在计算光照时，通常会使用纹理来代替漫反射的颜色。  
纹理和模型之间是通过UV坐标来绑定的，通过映射函数，将模型坐标转成UV坐标，然后取到对应坐标值的颜色作为模型纹素的颜色。  
#### 纹理动画
使用纹理动画可以代替复杂的粒子系统模拟各种动画效果，可以使用序列帧图像来实现爆照效果：  
[![KsPHqf.png](https://s2.ax1x.com/2019/10/27/KsPHqf.png)](https://imgchr.com/i/KsPHqf)  
使用UV动画实现滚动的背景：  
[![KsPqZ8.png](https://s2.ax1x.com/2019/10/27/KsPqZ8.png)](https://imgchr.com/i/KsPqZ8)  
本质上是通过内置的时间参数，通过时间来控制UV坐标的便宜或者读取UV坐标的位置，来实现一种动画效果。  
#### 法线纹理
法线纹理中存储的就是物体表面的法线方向，掌握了物体表面的法线方向，就可以根据法线来模拟出物体上凹凸不平的效果，这样可以使用较少的资源，而实现一些复杂的表面效果。  
![KrI5gs.png](https://s2.ax1x.com/2019/10/27/KrI5gs.png)  
#### 渐变纹理
使用渐变纹理可以控制漫反射光照的结果，如：Gooch等人在1998年提出一种基于冷到暖色调的着色技术，来得到一种插画风格的渲染效果，在这种渲染下，物体的轮廓线会更加明显，而且具有多种色调变化。  
[![KroexA.md.png](https://s2.ax1x.com/2019/10/27/KroexA.md.png)](https://imgchr.com/i/KroexA)  
#### 遮罩纹理
遮罩用以保护某些区域，使其免于某些修改，比如，一个模型中不想应用高光反射的地方，可以使用遮罩来避免，这样会使其看起来更加真实，避免某些地方显得太曝。  
[![KrogMR.md.png](https://s2.ax1x.com/2019/10/27/KrogMR.md.png)](https://imgchr.com/i/KrogMR)  
#### 渲染纹理
我们可以将摄像机渲染出的图像实时更新到渲染纹理中，然后通过这张渲染纹理来实现一些特殊的效果，如镜子效果：  
[![KsPEvt.png](https://s2.ax1x.com/2019/10/27/KsPEvt.png)](https://imgchr.com/i/KsPEvt)  
玻璃效果：  
[![KsPAgI.png](https://s2.ax1x.com/2019/10/27/KsPAgI.png)](https://imgchr.com/i/KsPAgI)  
#### 程序纹理
我们可以使用一些特定的算法来创建个性化图案，或者非常真实的自然元素，如木头、石子。使用程序纹理，我们可以通过各种参数来控制纹理的外观。  
[![KsPtbT.md.png](https://s2.ax1x.com/2019/10/27/KsPtbT.md.png)](https://imgchr.com/i/KsPtbT)  
#### 立方体纹理
立方体纹理的主要应用是天空盒和环境映射。  
天空盒是游戏中用于模拟背景的一种方法，使用时，整个场景被包围在一个立方体内，立方体的每个面使用立方体纹理映射技术进行映射。  
环境映射是由脚本渲染出物体所处环境的立方体纹理，然后可以实现反射和折射效果。  
[![KrObh6.png](https://s2.ax1x.com/2019/10/27/KrObh6.png)](https://imgchr.com/i/KrObh6)
[![KrOo7R.png](https://s2.ax1x.com/2019/10/27/KrOo7R.png)](https://imgchr.com/i/KrOo7R)
#### 深度和法线纹理
#### 噪声
### 后处理
屏幕后处理效果是游戏中实现屏幕特效的常见方法，Unity中提供了一个方便的接口：OnRenderImage，它有两个参数，Unity会把当前渲染得到的图像存储在第一个参数对应的源渲染纹理中，通过函数中的一系列操作后，把目标渲染纹理显示在屏幕上。然后利用Graphics.Blit函数完成对渲染纹理的处理。  
#### 调整屏幕亮度、饱和度、对比度
调整这些参数比较简单，只需要在片元着色器里，对一些参数进行调整即可。图中为调整后的效果。  
[![KysBjS.md.png](https://s2.ax1x.com/2019/10/27/KysBjS.md.png)](https://imgchr.com/i/KysBjS)  

#### 边缘检测
边缘检测是描边效果的一种实现方法，但精度不高。除此之外，在图像识别领域，边缘检测也多有应用。  
要实现边缘检测，首先要了解一个概念，卷积。  
卷积近年来在深度学习领域大放异彩，在图像处理领域其实也应用颇多，在图像处理中，其本质是一个滤波器，也叫边缘检测算子，常见的边缘检测算子有：Roberts、Prewitt、Sobel。  
[![Ky6NSP.md.png](https://s2.ax1x.com/2019/10/27/Ky6NSP.md.png)](https://imgchr.com/i/Ky6NSP)  
最终可以实现如下效果：  
[![Kysy7j.md.png](https://s2.ax1x.com/2019/10/27/Kysy7j.md.png)](https://imgchr.com/i/Kysy7j)  
只显示边缘的效果：  
[![Kys0c8.png](https://s2.ax1x.com/2019/10/27/Kys0c8.png)](https://imgchr.com/i/Kys0c8)

#### 高斯模糊
高斯模糊是很常见的模糊效果，与均值模糊和中指模糊相比，效果好一点，但是计算消耗要多很多。  
高斯模糊的实现也是基于卷积，只不过在计算时的算子略有不同，它使用一个正方形大小的滤波，中间的每个元素的计算都是基于高斯方程：
$$
G(x,y)=\frac{1}{2\pi \sigma^2}e^\frac{x^2+y^2}{2\sigma^2}
$$
一个5x5大小的高斯核，如图：  
[![Kyg5GR.md.png](https://s2.ax1x.com/2019/10/27/Kyg5GR.md.png)](https://imgchr.com/i/Kyg5GR)  
最终可以实现如图所示的效果：  
[![Kysrng.md.png](https://s2.ax1x.com/2019/10/27/Kysrng.md.png)](https://imgchr.com/i/Kysrng)

#### Bloom效果
这是一种可以模拟真实摄像机的一种图像效果，它可以让画面中较亮的区域“扩散”到周围的区域中，造成一种朦胧的效果，其实现的效果类似于HDR，但是资源消耗更少。  
它的原理也很简单，首先提取出屏幕中较亮的部分，然后对其进行高斯模糊，最后和原图像混合。可以得到如下效果：  
[![KyssBQ.md.png](https://s2.ax1x.com/2019/10/27/KyssBQ.md.png)](https://imgchr.com/i/KyssBQ)  

#### 运动模糊
摄像机曝光时，拍摄场景发生了变化，就会产生模糊的画面。而通过屏幕后处理，可以模拟这种效果，其中一种实现的方法，就是利用一块累积缓存，来混合多张连续的图像。  
[![KyscAs.md.png](https://s2.ax1x.com/2019/10/27/KyscAs.md.png)](https://imgchr.com/i/KyscAs)  

### 渲染优化
和PC平台相比，移动平台上的GPU架构有很大的不同。尤其是在Android平台上，不用设备的硬件，图形芯片、屏幕分辨率等，大相径庭。  
影响性能的因素主要可以分为两方面，CPU和GPU，在CPU中，过多的Draw Call和复杂的脚本和物理模拟会成为性能瓶颈。  
在GPU中，过多的顶点、过多的逐顶点计算，过多的片元和过多的逐片元计算也是性能的瓶颈。  
除此之外，使用尺寸很大且未压缩的纹理也会影响性能。  
在Unity中，使用动态批处理和静态批处理可以有效减少DrawCall。  
而想要减少顶点数目，则需要优化几何体以及遮挡剔除技术。  
减少片元数目，可以控制渲染顺序，减少实时光照和阴影等等。





