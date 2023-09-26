---
layout: post
title: "Unity动态图集实现"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - Sprite
  - Unity
---

最近在接入动态图集，因为局内DrawCall多的时候也有接近100，且局内引用了很多大图集，有很多图是用不上的，额外消耗了不少内存。  
### 动态图集实现
#### 整体思路
1.初始化创建一个Texture2D（TexAtlas）
2.预加载局内用到的图片（prefab动态图集组件上记录资源名，英雄头像、技能图标等），将其渲染到TexAtlas上
3.组件初始化、GetSprite时，从TexAtlas上找到对应的区域。

#### TexAtlas格式选择
+ no Mipmaps
+ filterMode选择Biliner，不选择Point，否则图片会有颗粒感，不平滑
+ TextureFormat RGBA32，如果是2048 * 2048的贴图就是占用32MB，但是Unity2021版本提供了压缩函数，可以压缩到8MB，所以这个大小就不算多。

#### 怎样渲染到TexAtlas
前期测试主要尝试了两种方法，第一种Graphics.CopyTexture，GPU级别的接口，在效率和和内存上是最优解，但是个别手机的GPU不支持这个接口，另外压缩后的图片在拷贝时也会出现一些问题，要求目标和源图格式相同，所以局限性较大，我们项目中的图集是经过压缩的格式，所以这条路行不通。

第二种方法Graphics.Blit，通过shader将源图片渲染到RenderTexture上，再将RenderTexture转为Texture2D（如果是RawImage就不用）,相比于第一种方法过程稍显复杂，开销较大，所以需要在Loading时进行预处理。但好处是可以定制整个渲染的过程，使得后面填满之后降分辨率复制的操作成为了可能，另外局限性较小，所以最终选择了这种方法。

另外还有两种方法：GL编程（Low-level graphics library），和Graphics.Blit差不多，但是实现起来比较麻烦。后面会用这种方式测试一下，和Graphics.Blit的效率进行对比。最后一种方法是Texture2D的Color[] GetPixels() 以及SetPixels，相对Graphics.Blit消耗小一点，但是要求源图片必须是可读写，这会导致内存占用两份，虽然会卸载，但我们局内也会引用大厅的图集，如Common图集，所以也不可取。

#### 矩形打包算法
矩形打包的思想是将较小的矩形尽可能紧密的放置在较大的矩形容器里，这在生成包含多个子纹理的大纹理时特别有用。我的实现在主矩形中使用了空闲矩形的概念。压缩矩形总是放置在某个空闲矩形的左上角，它们完全适合该矩形，为了获得非常接近最优的填充，选择填充矩形适合的最左空闲矩形的顶部进行放置。  
最初只有一个空闲矩形，就是整个矩形本身。将第一个矩形放进来后，移除空闲矩形栈中的第一个矩形，并生成两个新的空闲矩形（如果空间被占满，则不会生成新的矩形，如果高或宽被占满，则会生成一个矩形）。加入新矩形与这个流程是一样的，与新加入矩形相交的任何空闲矩形都会被切割成围绕这个矩形的新的较小的空闲矩形，然后需要删除由空闲矩形完全包裹着的空闲矩形。  
下图显示了空闲矩形的组合方式。
[![j3rgcd.png](https://s1.ax1x.com/2022/07/02/j3rgcd.png)](https://imgtu.com/i/j3rgcd)
左侧图像显示，放置第一个矩形后分割出了两个空闲矩形，放置第二个矩形会将空闲矩形2划分为两个新的空闲矩形，并将空闲矩形1划分为三个新的空闲矩形，但是因为其中有被完全包含在更大的空闲矩形中，因此被删除了。

算法源码：[RectanglePacking](https://github.com/villekoskelaorg/RectanglePacking)，作者：villekoskelaorg

C#版本：[UnityRuntimeSpriteSheetsGenerator](https://github.com/DaVikingCode/UnityRuntimeSpriteSheetsGenerator)

#### 预加载图片
【预加载】其实指的是将图片复制到动态图集的那张大图上。

在PreLoading中加入了一个PreloadImage的流程，在这个过程中需要收集ImageUI所使用的图片，主要分为两种：1，Prefab上挂载的图片；2，代码中动态加载的图片。收集完成后把它们一起复制到动态图集上。但是代码动态加载图片分散在各个代码文件中，收集起来并不简单。目前使用的方法是定义一个接口ISpriteCollect：
```C#
interface ISpriteCollect
{
    void CollectSprites(HashSet<string> spriteNames);
}
```
在需要动态加载图片的脚本里，实现这个接口，返回需要加载的图片名，这里以UIJumpBtnPanel为例：

```C#
public void CollectSprites(HashSet<string> spriteNames)
{
    spriteNames.Add("common_jump");
    var self = MatchStart.matchMembers[0];
    spriteNames.Add(MatchStart.Instance.GetIcon(self.HeroName, "PS", ""));
}
```
在预加载流程中，会遍历所有实现了这个接口的脚本，得到一个包含所有图片名字的集合。

#### 一些大单图的处理
一开始动态图集上会被添加一些显示频率很低的大图，占用了很大一部分空间，而对于这些大图，实际上并没有合批的需求（因为出现的频率非常低），所以在UIDynamicImage上增加了分组选项。对于设置为Single的图，会以单图的方式存放，可以避免出现图集不够用的情况。

#### GetSprite实现
DynamicImage继承自Image，需要显示时从DynamicAtlas中获取Texture以及对应图片在Texture上的区域，通过Sprite.Create方法创建对应的Sprite。如果是RawImage则比较简单，设置好Texture以及对应的位置大小即可。

### 结果
动态图集接入后，局内只需要一个图集即可，占用内存为8MB，相比于之前的56MB减少了48MB。DrawCall从100左右降到了50，之所以还有这么多，是因为还有除了图集之外的一些其他问题，这会在后面去尝试解决，争取最终能到个位数。