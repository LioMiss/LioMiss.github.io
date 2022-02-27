---
layout: post
title: "Unity接入Wwise热更新方法汇总"
subtitle: ''
author: "LioMiss"
header-style: text
tags:
  - Wwise
  - Unity
  - 热更新
---

Wwise是一款强大的音频中间件，是连接音频设计师与程序员的桥梁，为设计师提供了强大的音频功能与众多的效果器，为程序则提供了完善的功能接口与实用的调试工具，但在项目使用时还是会遇到各类问题，热更新正是其中之一，今天就来汇总一下Unity使用wwise后音频文件的热更新方法。

### 使用File Packager打包Pck文件（从Wwise2015开始）
这是Wwise最原始的热更新方式，因为这个方式在单机游戏上广泛应用，所以Wwise官方将其称为DLC，其兼容性也最强，可以说打Wwise支持Unity的那天起这个功能就存在了，所以不用担心自己的Wwise版本不支持的问题，只是不用版本的接口略有差异，好在官方有文档，参考文档使用不成问题。

这里只简要介绍一下工作流。

首先，音频设计师手动在File Packager中将需要更新的文件打包成Pck文件。

Pck文件随着游戏DLC更新文件下载解压到某目录。

调用AkSoundEngine.AddBasePath()添加DLC音频目录，随后调用AkSoundEngine.LoadFilePackage()加载。

这个工作流看起来很简单，后期更新甚至不需要程序做什么，但是每次更新需要生成Pck文件对于音频设计师而言是一个不小的负担，尤其是在迭代速度飞快的手游上，面对不同版本不同分支，对于版本管理工具操作本就不熟练的音频设计师来说十分容易出现错误。其更深层次原因其实是这个方法更适用于更新不频繁的单机游戏。

### 使用Addressables系统（Wwise2021）
unity2019之后引入了Addressable系统作为热更新解决方案，如果不深入使用的话它确实足够简单方便，但是一旦出现一些复杂情况它就会显得捉襟见肘，毕竟还处于实验阶段，很多功能尚不完善，不过在此不多赘述。

Wwise官方在2021版本中加入了对Addressables系统的支持，还为Unity开发了一个插件，用于管理Wwise文件的热更新：https://github.com/audiokinetic/WwiseUnityAddressables

这个插件需要unity2019以上的版本，Wwise2021以上版本，使用后可以将Wwise音频文件引入Addressable的打包系统，其工作流与Addressable本身的工作流基本一致，教程地址:https://www.audiokinetic.com/zh/library/edge/?source=Unity&id=pg_addressables.html

与制作pck的方式相比，这种方式音频设计师无感知，和Addressable兼容性好，但是有版本的硬性要求，另外Addressable本身有些坑还没填完，这个插件亦然，实际有多少坑，只有用了才知道。

### 以流媒体形式加载SoundBank
由于当时我们项目用了Addressable，所以我曾尝试在项目中使用Wwise的Addressable插件吗，但是因为我们的Wwise版本是2019，所以很多接口都对应不上，只好作罢。不过经过研究Wwise插件的代码，发现它的原理其实就是通过流的方式加载Soundbank，虽然在2019版本中接口参数有些不同，但稍作修改就可以使用了。

我们只需要将Soundbank文件存储为Addressable所支持的TextAsset，然后通过Addressable的加载接口加载文件，读取其二进制数据，这并不难做到。但Wwise引擎的接口需要我们传入这个bytes数组的指针，而且需要对原始数据做一些处理，处理的代码我直接参考Wwise插件，加载以及处理数据指针的代码如下：
```C#
public void LoadBankFromBytes(string bankName, Action callback)
{
    var path = "Assets/resourcesother/audio/generatedsoundbanks/" + bankName + ".bytes";
    NResourceManager.Instance.LoadResource<TextAsset>(path, (asset) =>
    {
        var ptr = AllocateAlignedBuffer(asset.bytes);
        AkSoundEngine.LoadBank(ptr, (uint)asset.bytes.Length, out uint out_bankId);
        callback?.Invoke();
    });
    callback?.Invoke();
}

private const long AK_BANK_PLATFORM_DATA_ALIGNMENT = AkSoundEngine.AK_BANK_PLATFORM_DATA_ALIGNMENT;
private const long AK_BANK_PLATFORM_DATA_ALIGNMENT_MASK = AK_BANK_PLATFORM_DATA_ALIGNMENT - 1;
private IntPtr AllocateAlignedBuffer(byte[] data)
{
    uint uInMemoryBankSize = 0;
    var array =
        System.Runtime.InteropServices.GCHandle.Alloc(data, System.Runtime.InteropServices.GCHandleType.Pinned);
    var pInMemoryBankPtr = array.AddrOfPinnedObject();
    uInMemoryBankSize = (uint)data.Length;
    if ((pInMemoryBankPtr.ToInt64() & AK_BANK_PLATFORM_DATA_ALIGNMENT_MASK) != 0)
    {
        var alignedBytes = new byte[data.Length + AK_BANK_PLATFORM_DATA_ALIGNMENT];
        var new_pinnedArray =
            System.Runtime.InteropServices.GCHandle.Alloc(alignedBytes, System.Runtime.InteropServices.GCHandleType.Pinned);
        var new_pInMemoryBankPtr = new_pinnedArray.AddrOfPinnedObject();
        var alignedOffset = 0;

        if ((new_pInMemoryBankPtr.ToInt64() & AK_BANK_PLATFORM_DATA_ALIGNMENT_MASK) != 0)
        {
            var alignedPtr = (new_pInMemoryBankPtr.ToInt64() + AK_BANK_PLATFORM_DATA_ALIGNMENT_MASK) &
                                ~AK_BANK_PLATFORM_DATA_ALIGNMENT_MASK;
            alignedOffset = (int)(alignedPtr - new_pInMemoryBankPtr.ToInt64());
            new_pInMemoryBankPtr = new System.IntPtr(alignedPtr);
        }

        System.Array.Copy(data, 0, alignedBytes, alignedOffset, data.Length);
        pInMemoryBankPtr = new_pInMemoryBankPtr;
        array.Free();
        array = new_pinnedArray;
    }
    return pInMemoryBankPtr;
}
```
所以最后的工作流就是这样，打包流程中复制Wwise音频文件到resourcesother目录下（热更新资源文件夹），需要修改文件的后缀名，随后进入Addressable正常的打包流程，然后在加载bank时采用上述流的方式加载。

一切都看似很完美，但是仅限于项目中没有使用流媒体文件(Wem格式)的情况，这取决于音频设计师，很不幸，我们项目就用了，所以我们现在还需要解决Wem文件的热更新以及加载问题。可惜的是，wem文件是在Wwise调用Soundbank时内部加载的，默认会去Wwise指定的几个资源目录加载，无法修改，至少我目前还没找到方法。

到这里一些仿佛又回到原点，如果我们想要继续使用Addressable的热更新方式去更新wem文件的话，就只能先类似于soundbank文件一样复制到resourcesother目录，然后在游戏运行时（或者解压更新包时）当做TextAsset加载二进制数据，并将其复制到额外目录（Application.persistentDataPath）修改后缀名为wem，最后调用AkSoundEngine.AddBasePath将此目录添加到wwise。

这样也可以，就是绕，仿佛绕这一圈就是为了用Addressable的更新流程，反倒不如自己做，所以才有了最后的解决方案，一切回归原始。

### 将SoundBank下载到额外目录
最后这种方案原理是最简单的，但是需要写不少代码，就是把它的资源文件都下载到额外目录（Application.persistentDataPath），通过对比md5，决定要下载哪些文件，所以还要在打包时生成一份md5列表，总的来说需要自制一套热更新的流程，但也因为是自制的，所以适配性更强，可以满足任何需求。

最后记得在代码中调用AkSoundEngine.AddBasePath添加额外目录到wwise。