---
layout: post
title: "UE项目符号服务器搭建"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - Unreal Engine
  - Unreal
  - Symbol Server
---

## 前言
在开发UE项目时，我们经常会遇到各种崩溃，要定位问题，就需要符号文件（PDB），简单的做法是：1.将符号文件打进包里； 2.提供符号文件的下载链接，需要时手动下载。但是这两种方法都有其缺点，第一种方法会导致包体积过大，第二种方法比较繁琐。其实除此之外，我们还可以搭建一个符号服务器，在VS或者Rider中调试Dump时，会自动从符号服务器下载匹配版本的符号文件，大大提高了调试效率。
### 上传符号文件
符号服务器实际并不是一个持续运行的服务，而是一个符号文件的存储空间，所以我们只需要将符号文件上传到一个地方即可。
比如可以在nas上指定一个文件夹专门存储符号文件。  
上传的命令如下：
```
symstore add /r /f .\YourProjectName\Binaries\Win64\YourProjectName*.pdb /s \\nas-url\Build-Dev\Debug\Symbols /t kajhsksa && .\tools\Others\symstore add /r /f .\YourProjectName\Binaries\Win64\YourProjectName*.exe /s \\nas-url\Build-Dev\Debug\Symbols /t kajhsksa
```
p.s. symstore是Wndows提供的工具，该工具位于[Debugging Tools for Windows](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/)包中。

看到日志：
```
SYMSTORE: Number of files stored = 2
SYMSTORE: Number of errors = 0
SYMSTORE: Number of files ignored = 0
Finding ID...  0000013504
```
说明文件上传成功。
随后可以在符号存储路径看到上传的符号文件：
[![pZJKHot.png](https://s41.ax1x.com/2025/12/25/pZJKHot.png)](https://imgchr.com/i/pZJKHot)
其中，那长串的文件夹名是这个Pdb的标识符，每个Pdb文件都有一个唯一的标识符，用于匹配符号文件。

### Visual Studio使用符号服务器
成功上传符号文件之后，我们就可以在VS中使用符号服务器了。  
打开VS，点击菜单栏的“调试”->“符号”->“符号文件位置”，点击右侧加号添加：
[![pZa3vh4.png](https://s41.ax1x.com/2026/01/04/pZa3vh4.png)](https://imgchr.com/i/pZa3vh4)
然后输入你的符号服务器地址，如：
```
\\nas-url\Build-Dev\Debug\Symbols
```
然后保存，调试Dump的时候，VS会自动从符号服务器下载匹配版本的符号文件，大大提高了调试效率。

### Rider使用符号服务器
和vs类似，Rider中也可以配置符号服务器地址：
[![pZa8Mut.png](https://s41.ax1x.com/2026/01/04/pZa8Mut.png)](https://imgchr.com/i/pZa8Mut)

### 源码服务器
符号服务器只能帮助我们下载匹配版本的符号文件，但是符号文件中并没有源码信息，如果想要通过Dump定位到源码，还需要源码服务器。
要使用源码服务器，首先要有一个版本控制系统，比如git，p4，svn等。
它的原理是将源码版本信息写入到符号文件中，在调试的时候会自动查找匹配的源码文件，从版本管理系统中下载对应的源码。

#### 将源码信息写入符号文件
这主要依赖于DebugTools里面的srcsrv，以p4为例，将源码信息写入到符号文件的命令如下：
```
tools\srcsrv\p4index.cmd -symbols=%system.teamcity.build.checkoutDir%\YourProjectName\Binaries\Win64 -source=%system.teamcity.build.checkoutDir% 
```

运行命令后，输出如下log，即表示成功：
```
17:36:06   ssindex.cmd [STATUS] : Server ini file: C:\teamcity-build\YourProjectName\tools\Others\srcsrv\srcsrv.ini
17:36:06   ssindex.cmd [STATUS] : Source root    : C:\teamcity-build\YourProjectName
17:36:06   ssindex.cmd [STATUS] : Symbols root   : C:\teamcity-build\YourProjectName\\\Binaries\Win64
17:36:06   ssindex.cmd [STATUS] : Control system : P4
17:36:06   ssindex.cmd [STATUS] : P4 program name: p4.exe
17:36:06   ssindex.cmd [STATUS] : P4 Label       : <N/A>
17:36:06   ssindex.cmd [STATUS] : Old path root  : <N/A>
17:36:06   ssindex.cmd [STATUS] : New path root  : <N/A>
17:36:06   ssindex.cmd [STATUS] : Partial match  : Not enabled
17:36:06   --------------------------------------------------------------------------------
17:36:06   ssindex.cmd [STATUS] : Running... this will take some time...
17:36:06   ssindex.cmd [STATUS] : GatherFileInformation, Server address is: 10.24.102.24:1666
17:36:18   ssindex.cmd [STATUS] : Save info to C:\teamcity-build\YourProjectName\\\Binaries\Win64\\Client.pdb
17:36:20   ssindex.cmd [STATUS] : Save info to C:\teamcity-build\YourProjectName\\\Binaries\Win64\\Server.pdb
17:36:20   ssindex.cmd [STATUS] : Save info to C:\teamcity-build\YourProjectName\\\Binaries\Win64\UnrealEditor-YourProjectName.pdb



```
参考资料：  
[Debug Symbols](https://learn.microsoft.com/en-us/windows/win32/dxtecharts/debugging-with-symbols?redirectedfrom=MSDN)  

