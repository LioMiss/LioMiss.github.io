---
layout: post
title: "HybridCLR评测"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - ECS
  - Unity
---
### 使用
#### 限制
+ 现在支持最好的是Unity2020系列，2019属于支持，但很快废弃的版本；2021版本Unity还有大量bug。
+ 必须使用IL2Cpp，且热更部分Api Compatibility Level只支持.Net 4.x
+ 不支持Use incremental GC
[![viKAsS.png](https://s1.ax1x.com/2022/07/30/viKAsS.png)](https://imgtu.com/i/viKAsS) 

#### 安装
[官方教程]("https://focus-creative-games.github.io/hybridclr/start_up/#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9")

安装完成后，与ILRuntime类似，需要通过LoadDll的方式来编译与加载热更新代码。  
会遇到和ILRuntime类似的代码裁剪问题，需要通过link.xml来避免。  
其他使用和原生无差异。

### 性能测试
| serial | language        | operate| time   |
| :----- | :-------------- | :------------ | :----- |
| 1      | C# Native        | Test1           | 11ms  |
| 2      | C# Native        | Test2           | 29ms   |
| 3      | C# Native        | Test3           | 7ms   |
| 3      | xlua             | Test1        | 430ms   |
| 4      | xlua             | Test2     | 800ms |
| 4      | xlua             | Test3     | 12371ms |
| 5      | HybridCLR          | Test1    | 344ms   |
| 6      | HybridCLR          | Test2    | 367ms  |
| 6      | HybridCLR          | Test3    | 201ms  |
| 7      | luajit jit mode | double mul    | 10ms   |
| 8      | lua native      | mul          | 100ms  |
| 9      | xLua            | fixed mul      | 5038ms |
| 10     | xLua            | native rshift | 90ms   |
| 11     | luajit          | int64 mul      | 1500ms |
| 12     | luajit          | ffi-int mul   | 1000ms |

经过测试发现，热更层代码在进行纯数值计算时性能比原生还是弱很多的，可能还是源于IL2Cpp的性能优势。相比于xlua优势也并不特别大。  
在外部代码调用上还是很有优势的，使用Unity内置类型参与计算，比xlua快很多，相比xlua是一个巨大的优势。  