---
layout: post
title: "基于UGUI的UI绑定代码生成工具"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - UGUI
  - Unity
  - C#
---

写过UI逻辑脚本的都知道，代码要引用UI预制件上的组件要么通过“手拖”的方式，要么在代码里通过transform.Find()的方式查找，其中，前者由于UI的预制件经常要被UI及动效的同事修改，可能会导致引用丢失，而且不易察觉，所以不常用，更多的时候还是通过代码中查找的方式，这样虽然也有路径修改或者结点结构修改导致找不到目标的风险，但是因为都在代码中，所以较为清晰，且一目了然。  
但正如前文所述，在代码中查找依然有几朵不小的乌云，其中之一就是UI或者动效人员可能会改变UI的结构，导致找不到目标，而且开发人员在写这类代码时往往很烦，因为它不需要动脑设计——属于重复性体力劳动，但同时还很伤神，因为UI结构往往总是嵌套很多层，需要一层一层对照着写。于是就有了工具的需求，也就是今天文章的主题，UI绑定代码生成工具。

### 使用方法
使用很简单，只需要右键点击要导出绑定代码的Prefab文件，找到Generator-UIBinder，点击即可。同时还有一个不生成子UI的代码的选项。  
导出完成后会打印导出成功的日志。 
导出的脚本存放在一个独立的UIBinder的文件夹下，以分布类的方式创建。和UI本身的逻辑代码分离，从而避免导出绑定代码时覆盖掉逻辑代码。  
至于哪些组件要导出，怎么导出，则由结点的名字根据导出规则来决定。 

### 导出规则
1.前缀为"m_"的节点会被自动导出。  
2.前缀为“UI”的节点会被当做一个独立的UI导出。  
3.前缀为"\_"的节点会被忽略。(包括它的子节点)  
4.如果该节点以组件名结尾，则生成的绑定代码为指定的组件，如：节点名为m_TitleText，那么就只会生成Text组件的绑定5.代码。否则会生成该节点上挂载的所有组件（除RectTransform）的绑定代码，如果只有RectTransform组件，则生成RectTransform的绑定代码。  
6.生成的字段名为“m_节点名+组件名”，如：节点名为m_Title，那么其Text组件的绑定代码命名就是m_TitleText，如果节点名为m_TitleText，生成的字段名也还是m_TitleText。  
7.同一个UI下前缀为"m_"或"UI"的节点名字不能相同，否则导出时会报错，满足规则4中“m_” + 组件名的节点除外。（p.s. 两个不同子UI下的节点可以重名）

组件列表
目前支持的组件以及其生成绑定代码或在节点名中指定导出某个组件时的命名如下表：
| 组件名 | 绑定代码命名/结点命名指定组件        |
| :----- | :-------------- | 
|NUIListView|ListView|
|NUIListItem|	ListItem|
|InputField	|InputField
|Dropdown	|Dropdown
|Slider	|Slider
|ToggleGroup	|ToggleGroup
|Toggle	|Toggle
|Button	|Button、Btn
|Image	|Image
|Text	|Text
|RectTransform	|Rect
|Transform	|Transform
|GameObject	|GameObject
|子UI：UIXXXX	|UIXXXX

如需添加组件类型，只需要在代码上加入对应的名字即可：
```c#
   // 默认导出组件
    static List<string> componentNames = new List<string>() {
        "NUIListView",
        "NUIListItem",
        "NCommonPanel",
        "RectTransform",
        "Animator",
        "UITweenPosition",
        "UITweenSlider",
        "EventTrigger",
        "InputField",
        "Dropdown",
        "Slider",
        "ToggleGroup",
        "Toggle",
        "Button",
        "Image",
        "RawImage",
        "Text",
    };

    // 通过名称指定导出的组件字典
    static Dictionary<string, string> componentFieldNames = new Dictionary<string, string>() {
        { "ListView","NUIListView"},
        { "NUIListView","NUIListView"},
        { "ListItem","NUIListItem"},
        { "NUIListItem","NUIListItem"},
        { "TweenPosition","UITweenPosition"},
        { "TweenSlider","UITweenSlider"},
        { "InputField", "InputField"},
        { "Dropdown", "Dropdown"},
        { "Slider","Slider" },
        { "ToggleGroup","ToggleGroup" },
        { "Toggle","Toggle" },
        { "Button","Button" },
        { "Btn","Button" },
        { "RawImage", "RawImage" },
        { "Image","Image" },
        { "Text","Text" },
        { "EventTrigger","EventTrigger" },
        { "Rect","RectTransform" },
        { "RectTransform","RectTransform" },
        { "Transform","Transform" },
        { "GameObject","GameObject" },
        { "Animation", "Animation" },
        { "Animator", "Animator" }
    };
```

### 总结
通过一定的命名规则导出绑定代码，在使用者熟练掌握这些规则后，在完成prefab基本结构的创建后，就可以放心的把它交给UI，以及动效人员，只需要双方对这个规则有一定了解并达成共识，就可以保证虽然节点的结构改变，但只需要重新导出一下代码就能使代码正常工作。这大大的提高了多人协作的效率。  
同时，对于程序员本身来说，可以省去很多的重复性或者反复的劳动，让我们心情更加舒畅的同时，也有更多的精力放在更有趣的事情上。  
代码仓库：https://github.com/LioMiss/UGUIBinderGenerator