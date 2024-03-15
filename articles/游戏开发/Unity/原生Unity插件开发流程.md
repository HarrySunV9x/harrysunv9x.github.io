---
title:原生Unity插件开发流程
date:2024-02-18
---

# 原生Unity插件开发流程

## 背景

Unity插件的开发可以被视为一个完整的C++项目，涵盖了从编码、编译到打包的整个过程。如果有点追求，还可以考虑加入自动化测试和发布流程。

本文将深入探讨Unity插件的开发流程，内容涉及上述所提到的代码的编写、编译、打包，以及使用Jetbrains进行持续集成（目前还在研究阶段）的相关步骤。

## 代码编写

之前总结过[外部库引用](https://harrysunv9x.github.io/blog/5aSW6YOo5bqT5byV55So)，当时便以Unity插件进行了举例。

### 插件源代码

原生插件的用法来自Unity官方文档：

https://docs.unity3d.com/cn/2022.3/Manual/LowLevelNativePluginProfiler.html

文档中举例了一个Profiler插件的使用方法，我们需要调用对应的API ，并进行实现。然后在Unity项目的CS脚本中通过外部库的方式引入这段代码。

#### 1. 头文件引用

Unity插件实际上是根据其提供的API将自定义代码插入到其源码的某个模块中。

以Unity的Profiler为例：

ProfilerManager负责管理每个Profiler打桩点（Marker）及其对应的Marker类。每个Marker类都含有一系列的回调函数（Callback）。

要自定义一个方法，首先需要创建一个回调函数。

接下来，使用Unity提供的RegisterMarkerEventCallback接口，将这个自定义的回调函数加入到相应Marker类的回调函数列表中。

当程序执行到某个打桩点时，它会遍历该点所有的回调函数，并调用包括我们自定义方法在内的这些函数。

在这个插件的实现中，Callback是我们自己写的，而想要让Callback在游戏里运行，就需要调用Unity自身的接口RegisterMarkerEventCallback了。

Unity提供了一系列接口供开发者使用，接口放在Unity 安装的 `<UnityInstallPath>\Editor\Data\PluginAPI` 文件夹中。我们在开发插件时，需要将这个头文件拿出来用。

> ├── PluginAPI/
> │   ├── IUnityEventQueue.h
> │   ├── IUnityGraphics.h
> │   ├── IUnityGraphicsD3D11.h
> │   ├── IUnityGraphicsD3D12.h
> │   ├── IUnityGraphicsMetal.h
> │   ├── IUnityGraphicsVulkan.h
> │   ├── IUnityInterface.h
> │   ├── IUnityLog.h
> │   ├── IUnityMemoryManager.h
> │   ├── IUnityProfiler.h
> │   ├── IUnityProfilerCallbacks.h
> │   ├── IUnityRenderingExtensions.h
> │   ├── IUnityShaderCompilerAccess.h
> │   └── LICENSE.md
> ├── CMakeLists.txt
> └── PluginsForUnity.cpp

#### 2. 插件源代码

我们写好自己的原生代码，如：

```C++
#include "PluginAPI/IUnityInterface.h"

extern "C" {
  UNITY_INTERFACE_EXPORT const char* UNITY_INTERFACE_API printHello() {
     return "Hello Plugin!";
  }
}
```

由于是原生实现，我们需要使用`extern "C"`指示编译器使用C语言的链接约定，确保原生库的函数可以被正确识别和调用。

UNITY_INTERFACE_EXPORT用于标记函数为公共API，以便从外部库中导出。

UNITY_INTERFACE_API定义了函数的调用约定。

在这里最好保持顺序，以免出错。

这个函数会在Unity的CS脚本调用printHello的时候打印Hello Plugin!

#### 3. CS脚本文件

插件源代码需要编译出一个Lib库文件，在CS脚本里引用。我们这里实现CS脚本文件：

```csharp
using UnityEngine;
using System;
using System.Collections;
using System.Runtime.InteropServices;
using UnityEngine.Rendering;

public class PluginsForUnity : MonoBehaviour
{
    // 导入在C++插件中定义的函数
    [DllImport("PluginsForUnity")]
    private static extern IntPtr printHello();

    void Start() {
        // 在游戏开始时打印消息
        Debug.Log(Marshal.PtrToStringAnsi(printHello()));
    }

    void Update() {
        // 每帧更新时再次打印消息
        Debug.Log(Marshal.PtrToStringAnsi(printHello()));
    }
}
```

[DllImport("PluginsForUnity")]以引用插件，下一行紧接一个引用的函数。

注意这里的返回类型需要使用IntPtr，以返回指针类型，注意关键词的顺序，避免出错。

## 库文件编译

为了实现跨平台编译，我们采用CMake+Ninja的编译方案（此处仅针对Windows端）。

### 1. 编写CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.26)
project(PluginsForUnity)

set(CMAKE_CXX_STANDARD 11)

add_library(PluginsForUnity SHARED
        PluginsForUnity.cpp)
```

PluginsForUnity为插件名，PluginsForUnity.cpp为源码文件名称。

### 2. 使用ninja进行编译打包

```shell
cmake -S . -B ./testBuild -G Ninja			# 在testBuild目录生成ninja配置文件
cd testBuild                      			# 进入testBuild目录
ninja -C .                        			# ninja编译文件
```

可以得到libPluginsForUnity.dll文件，将其移动到Unity项目的Asset文件夹中。

## CS脚本引入

将插件引入外部库，我们需要将刚刚写好的CS脚本绑定在某个Object上。

**1.创建Object：** 在Hierarchy窗口右键，Create Empty，创建一个空对象。

**2.绑定脚本：**选择这个空对象，将PluginsForUnity.cs拖到该对象的Inspector的空白处。

Unity会自动编译引入的脚本，如果有误，在Console会有红色字体提示。

另外，点击Asset目录的libPluginsForUnity.dll文件，可以对脚本引用的平台进行指定，本文仅使用Windows平台。

## 打包

为了将脚本供其他Unity应用开箱即用，我们需要将这个空对象以及脚本打包成unitypackage文件。

1. 将这个空对象转为预制体：在Hierarchy窗口拖动这个空对象到文件资源管理器中，如Asset文件夹。

2. 生成一个打包CS脚本，写入打包代码：

   ```csharp
   using UnityEditor;
   
   public class ExportUnityPackage
   {
       static void Export()
       {
           string[] assetPaths = { "Assets/EmptyObject.prefab" }; // 预制体的路径
           string packagePath = "Assets/PluginsForUnity.unitypackage"; // 生成的包的路径和文件名
           AssetDatabase.ExportPackage(assetPaths, packagePath, ExportPackageOptions.Recurse);
       }
   }
   ```

3. 命令行运行Unity编辑器进行的打包命令

   ```shell
   /path/to/Unity -quit -batchmode -projectPath /path/to/your/project -executeMethod ExportUnityPackage.Export
   ```

​	打包会在后台进行，看不到打包进度，打包成功会生成PluginsForUnity.unitypackage文件。

## Jetbrains持续集成

Todo..
