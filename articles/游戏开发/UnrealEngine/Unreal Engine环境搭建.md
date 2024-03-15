---
title: Unreal Engine 环境搭建指南
date: 2024-01-30
---

# Unreal Engine 环境搭建指南

## 背景

环境搭建一直都是一个十分恼人的问题。尤其是官网文档写的不够详细，又很难找到教程的情况下。本文将指导Unreal Engine 5.3的环境搭建，千里之行始于足下。

## 软件下载

Windows下载[visualstudio](https://visualstudio.microsoft.com/zh-hans/vs/)，本文是2022的版本。

Mac下载[Xcode](https://apps.apple.com/sg/app/xcode/id497799835?mt=12)，直接通过App Store下载。

Unreal是一款通过C++编译与程序编写的游戏引擎，因此需要C++编译器。普遍来说，在Windows系统中，常用的C++编译方式包括Visual Studio和mingw，但Unreal Engine只支持前者。对于Mac用户，Xcode是唯一的选择。

## IDE选择

本文选择Rider作为IDE。

当然，也可以直接用VS或者Xcode，甚至可以配置VSCode。但是我选择了[Rider](https://www.jetbrains.com/rider/)，实际体验不仅好用，跨平台体验也很好，进行相同的配置即可。

免责说明：

本文仅供学习使用，切勿进行任何商业用途！

如果有资金，请一定购买正版！

**学习版**

截止目前，学习版有两种方法，一种是已经挂掉的github仓库破解：

1、打开网址：https://3.jetbra.in，这里会检索可访问的破解密钥地址，选择任意一个进去即可。

2、下载破解文件：[jetbra.zip](https://hardbin.com/ipfs/bafybeia4nrbuvpfd6k7lkorzgjw3t6totaoko7gmvq5pyuhl2eloxnfiri/files/jetbra-ded4f9dc4fcb60294b21669dafa90330f2713ce4.zip)。

3、解压到某个位置。

4、对于Windows，直接执行jetbra/script/install-all-users.vbs即可；

5、对于Mac，终端运行sudo sh install.sh。经我测试mac还需要在执行后，将jetbra/vmoptions/rider.vmoptions拷贝到rider的目录去，如：~/Library/Application\ Support/JetBrains/Rider2023.2/，其他jetbrain的学习版理论上同理。

5、拷贝刚刚密钥地址的密钥，输入到Rider激活时到Activation Code里去。

6、激活成功。

## 项目编译验证——创建Character并添加摄像机

### 安装和初始化

1. **安装Epic Games Launcher和Unreal Engine**：
   - 下载并安装Epic Games Launcher。
   - 通过Launcher安装Unreal Engine。本教程以5.3版本为例。
2. **创建新项目**：
   - 启动Unreal Engine，选择“创建空白项目”。
   - 为项目命名时避免使用“test”，因为这是一个保留关键字，可能导致编译错误。

### 设置C++类

1. **创建C++ Character类**：
   - 在项目中，创建一个新的C++类。
   - 选择“角色（Character）”作为父类。
   - 将类命名为“MyCharacter”。
2. **添加蓝图类**：
   - 在资源管理器中，创建一个新的蓝图类。
   - 选择刚创建的“MyCharacter”作为父类。
   - 将蓝图类命名为“PlayerCharacter”，并保存。
3. **配置场景**：
   - 创建一个新的空白场景。
   - 将“PlayerCharacter”蓝图拖拽到场景中。

### 编写和配置代码

1. **在MyCharacter.h中添加摄像机支持**：

   ```c++
   #include ...
   class UCameraComponent;
   class USpringArmComponent;
   
   UCLASS()
   ...
   
   protected:
   	// Called when the game starts or when spawned
   	virtual void BeginPlay() override;
   	UCameraComponent *CameraComp;
   	USpringArmComponent *SpringArmComp;
   	
   ...
   ```

2. **在MyCharacter.cpp中初始化摄像机组件**：

   ```c++
   #include "FirstCharacter.h"
   
   #include "Camera/CameraComponent.h"
   #include "GameFramework/SpringArmComponent.h"
   
   // Sets default values
   AFirstCharacter::AFirstCharacter()
   {
    	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
   	PrimaryActorTick.bCanEverTick = true;
   	CameraComp = CreateDefaultSubobject<UCameraComponent>("CameraComp");
   	SpringArmComp = CreateDefaultSubobject<USpringArmComponent>("SpringArmComp");
   
   }
   ...
   ```

3. **编译并验证**：

   - 在Rider中，点击工具栏中的“Build and Reload”锤子图标。
   - 切换回Unreal Editor，选择刚才的蓝图类。
   - 如果场景中出现了摄像机视图，则表示代码编译成功，环境搭建完成。



> ## 命名规范
>
> - 所有代码和注释都应采用美式标准英语的拼写和语法。
>
> - 命名（如类型或变量）中的每个单词需大写首字母，单词间通常无下划线。例如：`Health` 和 `UPrimitiveComponent`，而非 `lastMouseCoordinates` 或 `delta_coordinates`。
>
> - 类型名前缀需使用额外的大写字母，用于区分其和变量命名。例如：`FSkin` 为类型名，而 `Skin` 则是 `FSkin` 的实例。
>
>   - 模板类的前缀为T。
>
>   - 继承自 `UObject` 的类前缀为U。
>
>   - 继承自 `AActor` 的类前缀为A。
>
>   - 继承自 `SWidget` 的类前缀为S。
>
>   - 抽象界面类的前缀为I。
>
>   - 列举的前缀为E。
>
>   - 布尔变量必须以b为前缀（例如 `bPendingDestruction` 或 `bHasFadedIn`）。
>
>   - 其他多数类均以F为前缀，而部分子系统则以其他字母为前缀。
>
>   - Typedefs应以任何与其类型相符的字母为前缀：若为结构体的Typedefs，则使用F；若为 `Uobject` 的Typedefs，则使用U，以此类推。
>
>     - 特别模板实例化的Typedef不再是模板，并应加上相应前缀，例如：
>
>       ```
>       typedef TArray<FMytype> FArrayOfMyTypes;
>       ```
>
>   - C#中省略前缀。
>
>   - 多数情况下，UnrealHeaderTool需要正确的前缀，因此添加前缀至关重要。
>
> - 类型和变量的命名为名词。
>
> - 方法名是动词，以描述方法的效果或未被方法影响的返回值。