---
layout: post
title: "UE5世界分区"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - Unreal Game Sync
  - Unreal
---

### Horde
Horde是UE5推出的提升工作流的服务框架，集成了原本的UBA，UGS元数据服务器，Build Graph等功能，简化了部署的工作，并且提供了更多的功能。
Horde官方文档：[Horde](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/horde-in-unreal-engine)
Horde提供以下功能，其中大部分可以独立启用或禁用：
+ 远程执行 ：该功能可将计算工作分摊到其他计算机，包括使用虚幻构建加速器进行C++编译。
+ 构建自动化(CI/CD) ：一种构建自动化系统，为使用大型Perforce仓库的团队设计。
+ 测试自动化 ：用于跨流和项目查询自动化结果的前端，与自动化工具和Gauntlet集成。
+ Studio分析 ：从虚幻编辑器接收遥测并显示关键工作流程指标图表。
+ UnrealGameSync元数据服务器 ：为使用UnrealGameSync的团队提供的各种功能，包括构建状态报告、评论聚合和众包构建健康功能。
+ 移动/主机设备管理器 ：一种用于分配和管理大量开发工具包和移动设备的系统。

### UGS
UGS是便于项目内部同步编辑器版本的工具，详见：
[UGS](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/unreal-game-sync-ugs-for-unreal-engine?application_version=5.5)

### 使用Horde作为UGS元数据服务器
UGS原本的元数据服务器部署比较麻烦，UE5.3中，由于Horde集成了UGS元数据服务器，所以可以使用Horde作为UGS的元数据服务器，只需要修改一下配置文件即可。
1.修改UGS的Deployment.json
设置正确的Horde Server地址，如：
```json
"HordeUrl":"http://127.0.0.1:13340",
"HordeToolId":"ugs-win",
"ApiUrl":"http://127.0.0.1:13340/ugs",
```

其中127.0.0.1可以替换为自己的服务器地址

2.启动Horde服务器
参考Horde官方文档，下载安装Horde Server即可，比较傻瓜，这里不再赘述。

3.写Post请求
原本的PostBadgeStatus.exe不支持HordeServer，需要自己写Post请求。  
如果不需要打标签，可以不用自己写post请求，只修改1中的地址就够了，如果需要打标签，更新版本的状态显示等，则需要自己写post请求：
以Postman为例，选择POST，输入地址：http://127.0.0.1:13340/ugs/api/metadata  
Headers中需要加入Content-Type:application/json，否则会报错  
Body中输入json，如：
```json
{
    "Stream": "//stream/branchName/",
    "Change": "123456",
    "Project": "YourProjectName",
    "UserName": "UserName",
    "Badges": [
        {
            "Name": "Editor",
            "State": "Success"
        }
    ]
}
```

4.启动UGS
在虚幻编辑器中启动UGS，可以添加Good或Bad标记，可以显示打好的标签。实现类似官方文档中的效果：

[![ugs界面](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/f13f0559-6a76-4c3a-816b-b317607d0719/ugs-changelist-context-menu.png)](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/f13f0559-6a76-4c3a-816b-b317607d0719/ugs-changelist-context-menu.png)