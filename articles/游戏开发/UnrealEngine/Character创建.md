---
title: Character创建
date: 2024-02-01
---

# Character创建

## 层次结构

创建一个Character类，我们可以看到头文件里声明是这样的：

```C++
class PROJECTNAME_API ASCharacter : public ACharacter
```

根据Unreal的规则，继承自AActor的类，需要以A开头命名。

我们一层层往上找，可以找到一个Character的层次结构：

```
UObjectBase
├── UObjectBaseUtility
│       └── UObject
│    │       └── AActor
│    │       │   ├── APawn
│    │       │   │   └── ACharacter
```

查询他们的文档：

**UObjectBase**：UObject的低级实现，根据官方API，这个类不应直接在游戏代码中使用，我们不用关心。

**UObjectBaseUtility**：提供UObject的实用功能，同样，这个类不应该直接使用。

**UObject**：不难理解，所有UE对象的基类。包含了8071个子类（5.3），怪不得GPT无法告诉我全部的内容。

**AActor**：游戏世界中所有可放置对象的基类。

**APawn**：可以被玩家或AI操控的角色的基类。

**ACharacter**：具有网格（mesh）、碰撞（collision）和内置移动逻辑的Pawns（角色）。

为了更好的理解Unreal的这些内容，我们通过官方文档[Gameplay Framework](https://docs.unrealengine.com/5.3/en-US/gameplay-framework-in-unreal-engine/)进行进一步了解。

## Gameplay Framework


虚幻引擎中的游戏框架提供了多个类和组件，可用作项目的构建模块。

1. **Actors（角色）**：Actor可能包含一系列Actor组件的集合，这些组件用于控制Actor的移动方式和渲染方式。在游戏过程中，Actor支持跨网络复制属性和函数调用。
2. **Cameras（摄像机）**：摄像机代表玩家的视角，即玩家如何看世界。PlayerController指定了一个摄像机类并实例化一个Camera Actor，用于计算玩家从哪个位置和方向看世界。
3. **Pawn（角色）**：Pawn类是可以由玩家或人工智能控制的所有角色的基类。Pawn是玩家或AI实体在世界中的物理表示。Character是Pawn的一种特殊类型，具有四处走动的能力。默认情况下，控制器（Controller）和Pawn之间是一对一的关系，这意味着每个控制器一次只能控制一个Pawn。
4. **Controllers（控制器）**：Controllers是非物理的Actors，可以控制Pawn或Pawn派生类，如Character以控制其动作。玩家控制器用于玩家控制Pawn，而AI控制器实现了它们控制的Pawn的人工智能。控制器使用Possess函数来控制Pawn，并使用UnPossess函数放弃对Pawn的控制。
5. **Gameplay Timers（游戏时间器）**：Gameplay Timers创建异步回调到特定的函数指针，触发在延迟后或一段时间内执行的事件。
6. **GameMode（游戏模式）**：游戏框架的基础是GameMode。在初始化关卡进行游戏时，会实例化一个AGameModeBase角色。GameMode设置了游戏的规则，它仅在服务器上实例化，不会存在于客户端。
7. **Game Features和Modular Gameplay插件**：这些插件帮助开发人员为其项目创建独立的功能。这些插件使项目的代码库保持干净可读，避免不相关功能之间的意外交互或依赖关系。
8. **用户界面（UI）和抬头显示（HUD）**：这是向玩家提供关于游戏信息的一种方式，有时允许玩家与游戏进行交互。

## 角色（Acter的生命周期）

参考文档：[Actor Lifecycle](https://docs.unrealengine.com/5.3/en-US/unreal-engine-actor-lifecycle/)

本节将介绍：

角色是如何被实例化或生成到关卡中的，包括角色的初始化过程。

角色如何被标记为待销毁（PendingKill），然后通过垃圾收集（Garbage Collection）被移除或销毁。

![img](https://docs.unrealengine.com/5.3/Images/making-interactive-experiences/interactive-framework/actors/actor-lifecycle/ActorLifeCycle1.png)

### **从磁盘加载**（深蓝色）

1. **UEngine::LoadMap 或 UWorld::AddToWorld**：从磁盘加载角色
2. **PostLoad**：加载对象后立即执行，释放资源、重置状态或执行其他必要的操作。只用于加载后，不用于创建后，主线程执行。
3. **UAISystemBase::InitializeActorsForPlay**：在世界初始化所有角色并准备开始游戏时调用。
4. **ULevel::RouteActorInitialize**：根据不同的RouteActorInitializationState状态，遍历Actors执行以下不同的函数。
   - AActor::PreInitializeComponents：在组件初始化之前调用。
   - UActorComponent::InitializeComponent：在 BeginPlay之前初始化组件
   - AActor::PostInitializeComponents ：在其所有组件被初始化自定义初始化<-我的理解是将配置与自定义分开，避免出错
5. **AActor::BeginPlay**：可重写的本地事件，用于当这个角色开始播放时。

### 编辑器内播放（黄色）

在编辑器中播放路径中，角色是从编辑器中复制而不是从磁盘加载的。复制的角色以类似于“从磁盘加载路径”中描述的流程进行初始化。

## SCharacter.h

```c++
//以开源项目ActionRoguelike的角色代码为例说明，包含角色移动、视角控制、属性设置与交互等功能
//同时与初始文件对比，对默认代码与新增代码作出说明
#pragma once									// 防止重复包含，默认

#include "CoreMinimal.h"						// 引入核心最小依赖，默认
#include "GameFramework/Character.h"			// Character类，默认

// Included for struct FInputActionInstance (Enhanced Input)
#include "InputAction.h"						// 引入增强输入系统，新增
#include "SCharacter.generated.h"				// 自动生成的UClass头文件，默认

//引入Unreal标准类，新增
class UInputMappingContext;						// 输入映射
class UCameraComponent;							// 摄像机
class USpringArmComponent;						// 弹簧臂
class USInteractionComponent;					// 交互
class UAnimMontage;								// 动画蒙太奇
class USAttributeComponent;						// 属性
class UParticleSystem;							// 粒子系统
class USActionComponent;						// 动作系统

UCLASS()										// 定义一个新的UClass，ASCharacter，继承自ACharacter，默认
class ACTIONROGUELIKE_API ASCharacter : public ACharacter
{
	GENERATED_BODY()							// 宏，用于生成标准构造函数和序列化代码，暂时未找到官方说明与源码，默认

protected:
	
    // 定义输入映射和相关输入动作，新增
    
	/* EditDefaultsOnly 表示该成员变量可以在编辑器中修改默认值，但在游戏运行时无法修改。 */
    // 默认输入映射上下文对象，定义在何种情况下某些输入事件应该被触发
    UPROPERTY(EditDefaultsOnly, Category = "Input")			//UPROPERTY：属性成员变量；EditDefaultsOnly：在编辑器中修改默认值
    TObjectPtr<UInputMappingContext> DefaultInputMapping;   //指向 UInputMappingContext 类型的智能指针（smart pointer）变量。

    // 用于处理角色移动的输入动作对象
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    TObjectPtr<UInputAction> Input_Move;

    // 用于处理鼠标视角控制的输入动作对象
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    TObjectPtr<UInputAction> Input_LookMouse;

    // 用于处理手柄（游戏手柄）视角控制的输入动作对象
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    TObjectPtr<UInputAction> Input_LookStick;

    // 角色跳跃的输入动作对象
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    TObjectPtr<UInputAction> Input_Jump;

    // 与物体互动的输入动作对象
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    TObjectPtr<UInputAction> Input_Interact;

    // 角色奔跑的输入动作对象
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    TObjectPtr<UInputAction> Input_Sprint;

    // 角色冲刺的输入动作对象
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    TObjectPtr<UInputAction> Input_Dash;

    // 角色主要攻击的输入动作对象
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    TObjectPtr<UInputAction> Input_PrimaryAttack;

    // 角色次要攻击的输入动作对象
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    TObjectPtr<UInputAction> Input_SecondaryAttack;
	
	/* VisibleAnywhere 表示该成员变量可以在编辑器中查看，但不可以在编辑器中修改。 */
	//UPROPERTY(VisibleAnywhere, Category = "Effects")
	//FName TimeToHitParamName;

	/* 覆盖材质（Overlay material）通常用于在对象上添加额外的效果，比如伤害闪烁或高亮。这些效果可能需要一些额外的数据，例如颜色或透明度值。 */
    /* 这些额外的数据通常存储在一个集合中，例如一个数组或列表。这个集合中的每个数据都有一个唯一的索引，用来标识它在集合中的位置。 */
    /* 在代码中或材质中使用的索引值必须与覆盖材质中使用的自定义数据的索引相匹配。 */
    // 角色受击时的视觉效果索引，可在编辑器中查看
    UPROPERTY(VisibleAnywhere, Category = "Effects")
    int32 HitFlash_CustomPrimitiveIndex;

    // 摄像机弹簧臂组件的指针
    UPROPERTY(VisibleAnywhere)
    TObjectPtr<USpringArmComponent> SpringArmComp;

    // 摄像机组件的指针
    UPROPERTY(VisibleAnywhere)
    TObjectPtr<UCameraComponent> CameraComp;

    // 与互动相关的组件的指针
    UPROPERTY(VisibleAnywhere)
    TObjectPtr<USInteractionComponent> InteractionComp;

    // 角色属性组件的指针
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    TObjectPtr<USAttributeComponent> AttributeComp;

    // 角色动作组件的指针
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    TObjectPtr<USActionComponent> ActionComp;

	// Enhanced Input
	// Three parameter options available (FInputActionInstance, FInputActionValue, or none)
	
    // 定义移动和查看相关的方法，新增

    // 处理角色移动的方法
    void Move(const FInputActionInstance& Instance);			//FInputActionInstance表示输入动作（Input Action）的实例，包含：
    															//输入动作的名称或标识符，以指示是哪个动作被触发。
                                                                //输入动作的状态，例如按下、释放、持续按下等。
                                                                //触发输入动作的玩家控制器或设备。
                                                                //输入动作的时间戳，以确定动作何时触发。

    // 处理鼠标视角控制的方法
    void LookMouse(const FInputActionValue& InputValue);		//FInputActionValue表示输入事件的值

    // 处理手柄视角控制的方法
    void LookStick(const FInputActionValue& InputValue);

    // 角色开始奔跑的方法
    void SprintStart();

    // 角色停止奔跑的方法
    void SprintStop();

    // 角色进行主要攻击的方法
    void PrimaryAttack();

    // 角色进行黑洞攻击的方法
    void BlackHoleAttack();

    // 角色进行冲刺的方法
    void Dash();

    // 角色进行主要互动的方法
    void PrimaryInteract();

    // 定义健康变化时的回调函数，新增
	UFUNCTION()													//声明可在蓝图中调用。
	void OnHealthChanged(AActor* InstigatorActor, USAttributeComponent* OwningComp, float NewHealth, float Delta);

    // 重写PostInitializeComponents方法，新增
	virtual void PostInitializeComponents() override;			//虚函数，用于在 Actor 的组件初始化之后执行一些额外的自定义初始化工作。

    // 重写GetPawnViewLocation方法，新增
	virtual FVector GetPawnViewLocation() const override;		//虚函数，用于获取角色（Pawn）的观察位置（View Location）。

    // 定义寻找准星目标的方法，新增
	void FindCrosshairTarget();

    // 定义完成准星追踪的回调函数，新增，需要注意的时候，Character默认指挥覆写BeginPlay()，本类没有此覆写。
	void CrosshairTraceComplete(const FTraceHandle& InTraceHandle, FTraceDatum& InTraceDatum);

    // 定义追踪句柄，新增
	FTraceHandle TraceHandle;
	
public:	

    // 构造函数，默认
	ASCharacter();

    // 重写SetupPlayerInputComponent方法，默认，用于设置玩家输入控制器（Player Input Controller）的输入映射
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override; //

    // 重写Tick方法，默认，每一帧执行
	virtual void Tick(float DeltaSeconds) override;

    // 定义自我治疗的方法，新增
	UFUNCTION(Exec)
	void HealSelf(float Amount = 100);

private:
	// 定义是否有准星目标的标志，新增
	bool bHasPawnTarget;
};
```

### UPROPERTY说明

#### 定义和参数

`UPROPERTY` 是 Unreal Engine 中的一个宏，用于在 C++ 中声明类的属性。它的主要作用是在引擎的编辑器中暴露这些属性，并提供序列化、网络复制等功能。通过使用 `UPROPERTY`，开发者可以在编辑器中轻松地修改这些属性，而无需重新编译代码。此外，它还能确保属性在游戏过程中的持久性以及在多玩家游戏中的正确同步。

`UPROPERTY` 宏可以接受多个参数：

1. **可见性和编辑**
   - `VisibleAnywhere`：属性在属性窗口中可见，但不可编辑。
   - `VisibleInstanceOnly`：只在实例中可见。
   - `VisibleDefaultsOnly`：只在默认值中可见。
   - `EditAnywhere`：属性在属性窗口中可见并且可编辑。
   - `EditInstanceOnly`：只在实例中可编辑。
   - `EditDefaultsOnly`：只在默认值中可编辑。
2. **分类**
   - `Category="分类名"`：指定属性在编辑器中的分类。
3. **网络**
   - `Replicated`：使属性在网络游戏中被复制。
   - `ReplicatedUsing=函数名`：指定当属性通过网络更新时要调用的函数。
4. **序列化和保存**
   - `Transient`：属性不会被序列化，每次游戏启动时都会重置。
   - `BlueprintReadOnly`：属性在蓝图中只读。
   - `BlueprintReadWrite`：属性在蓝图中可读写。
   - `SaveGame`：属性会被包含在保存游戏数据中。
5. **高级**
   - `AdvancedDisplay`：属性在编辑器中默认折叠，需要展开才能看到。
   - `NoClear`：在编辑器中阻止将属性设置为“无”或“null”。
6. **元数据**
   - `meta=(关键字="值")`：为属性提供额外的元数据，如工具提示、范围限制等。
7. **数组相关**
   - `ArrayClamp=变量名`：使用另一个属性的值来限制数组的大小。

#### Catergory参数

```
    /**	UPROPERTY的Category类型：
     * Input（输入）：用于包含与输入事件、控制和用户交互相关的成员，例如输入动作、按键映射等。
     * Movement（移动）：用于包含与角色、物体或者移动相关的成员，例如速度、方向、加速度等。
     * Combat（战斗）：用于包含与战斗、攻击、伤害和武器相关的成员，例如攻击力、生命值、伤害函数等。
     * UI（用户界面）：用于包含与用户界面、HUD（头顶显示）和菜单相关的成员，例如分数、计时器、按钮等。
     * Physics（物理）：用于包含与物理模拟、碰撞检测和刚体动力学相关的成员，例如质量、碰撞框、力和重力等。
     * Sound（声音）：用于包含与声音效果、音乐、音频和音效相关的成员，例如音频剪辑、音量、声音效果等。
     * Effects（效果）：用于包含与视觉效果、特效和粒子系统相关的成员，例如粒子效果、光照、材质等。
     * Components（组件）：用于包含与类的组件、子对象或子系统相关的成员，例如摄像机组件、光源组件、碰撞体组件等。
     * Debug（调试）：用于包含用于调试和开发的成员，例如日志、调试标志、性能分析等。
     * Settings（设置）：用于包含用于配置游戏或角色的设置和选项，例如画面设置、音效设置、控制设置等。
     */
```

每种 `UPROPERTY` 类别的简单场景：

1. **Input（输入）**
   
   ```c++
   UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Input")
   float TurnRate;
   
   void TurnAtRate(float Rate)
   {
       // 使用TurnRate来处理角色的旋转
       AddControllerYawInput(Rate * TurnRate * GetWorld()->GetDeltaSeconds());
   }
   ```
   
2. **Movement（移动）**
   ```c++
   UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Movement")
   FVector CurrentVelocity;
   
   void MoveForward(float Value)
   {
       // 更新CurrentVelocity基于前进方向和输入值
       FVector Direction = GetActorForwardVector();
       CurrentVelocity = Direction * Value;
   }
   ```

3. **Combat（战斗）**
   ```c++
   UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category="Combat")
   int Health;
   
   void ReceiveDamage(int DamageAmount)
   {
       // 根据受到的伤害减少Health
       Health -= DamageAmount;
   }
   ```

4. **UI（用户界面）**
   ```c++
   UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="UI")
   FString PlayerName;
   
   void UpdatePlayerNameUI()
   {
       // 更新UI显示玩家名字
       MyPlayerHUD->SetPlayerNameText(PlayerName);
   }
   ```

5. **Physics（物理）**
   ```c++
   UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Physics")
   float Mass;
   
   void CalculateForce(FVector Direction, float ForceMagnitude)
   {
       // 根据物体的质量计算作用力
       FVector Force = Direction * ForceMagnitude / Mass;
       AddForce(Force);
   }
   ```

6. **Sound（声音）**
   ```c++
   UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Sound")
   USoundBase* FireSound;
   
   void PlayFireSound()
   {
       // 播放射击声音
       UGameplayStatics::PlaySoundAtLocation(this, FireSound, GetActorLocation());
   }
   ```

7. **Effects（效果）**
   ```c++
   UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Effects")
   UParticleSystem* ExplosionEffect;
   
   void TriggerExplosion()
   {
       // 触发爆炸效果
       UGameplayStatics::SpawnEmitterAtLocation(GetWorld(), ExplosionEffect, GetActorLocation());
   }
   ```

8. **Components（组件）**
   ```c++
   UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Components")
   UCameraComponent* CameraComponent;
   
   void ZoomIn()
   {
       // 使用摄像机组件进行缩放
       CameraComponent->SetFieldOfView(45.0f);
   }
   ```

9. **Debug（调试）**
   ```c++
   UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Debug")
   bool bShowDebugInfo;
   
   void ToggleDebugDisplay()
   {
       // 切换调试信息的显示
       bShowDebugInfo = !bShowDebugInfo;
   }
   ```

10. **Settings（设置）**
    ```c++
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Settings")
    float SoundVolume;
    
    void AdjustSoundVolume(float Volume)
    {
        // 根据用户设置调整声音音量
        SoundVolume = Volume;
        AudioComponent->SetVolumeMultiplier(SoundVolume);
    }
    ```

## SCharacter.cpp

### 头文件

```C++
#include "SCharacter.h" 								// 包含 SCharacter 类的头文件

#include "GameFramework/SpringArmComponent.h" 			// 包含弹簧臂组件的头文件
#include "Camera/CameraComponent.h" 					// 包含摄像机组件的头文件
#include "DrawDebugHelpers.h" 							// 包含调试帮助函数的头文件
#include "GameFramework/CharacterMovementComponent.h" 	// 包含角色移动组件的头文件
#include "SInteractionComponent.h" 						// 包含交互组件的头文件，自定义
#include "SAttributeComponent.h" 						// 包含属性组件的头文件，自定义
#include "SActionComponent.h"						 	// 包含动作组件的头文件，自定义
#include "Components/CapsuleComponent.h" 				// 包含胶囊碰撞组件的头文件
#include "SharedGameplayTags.h" 						// 包含共享游戏标签的头文件，自定义

// Enhanced Input
#include "EnhancedInputComponent.h" 					// 包含增强输入组件的头文件
#include "EnhancedInputSubsystems.h" 					// 包含增强输入子系统的头文件
#include "SPlayerController.h" 							// 包含 SPlayerController 类的头文件，自定义

#include UE_INLINE_GENERATED_CPP_BY_NAME(SCharacter) 	// 将类的实现嵌入到头文件中，提升编译速度
```

### 构造函数

角色的构造函数是在创建角色对象时首先被调用的函数，它用于进行角色对象的基本初始化工作。通常，角色的构造函数会执行以下任务：

1. **组件的创建和初始化：** 创建并初始化角色所需的组件，例如摄像机组件、碰撞组件、动画组件等。
2. **属性和状态的设置：** 初始化角色的属性，例如生命值、能量、速度等。
3. **绑定输入和动作：** 绑定输入和动作事件。  
4. **加载资源和设置外观：** 加载游戏资源，例如模型、纹理、声音等。
5. **初始化游戏逻辑：** 初始化中设置和启动特定的游戏逻辑或行为。
6. **与其他系统的交互：** 与其他游戏系统的连接和配置。

```C++
ASCharacter::ASCharacter()
{
    // 启用角色的主循环（Tick）功能，使其可以在每一帧中执行逻辑
    PrimaryActorTick.bCanEverTick = true;

    // 创建并初始化 SpringArm 组件
    SpringArmComp = CreateDefaultSubobject<USpringArmComponent>("SpringArmComp");	
    												// CreateDefaultSubobject 用于在构造函数中创建和初始化默认子对象。
    SpringArmComp->bUsePawnControlRotation = true; 	// 是否跟随角色旋转
    SpringArmComp->SetupAttachment(RootComponent); 	// SetupAttachment 通常用于将一个组件附加（连接）到另一个组件上，以构建游戏对象的层次结构
    SpringArmComp->SetUsingAbsoluteRotation(true); 	// 使用绝对旋转，而不受角色旋转的影响
    												// 弹簧臂的选择已经通过bUsePawnControlRotation设置为角色跟随
    												// 但是，角色的 CapsuleComponent 旋转时，这种旋转会暂时影响弹簧臂，直到稍后自我纠正
    												// 禁用相对旋转，以保证状态稳定

    // 创建并初始化 Camera 组件，将其附加到 SpringArm
	CameraComp = CreateDefaultSubobject<UCameraComponent>("CameraComp");
	CameraComp->SetupAttachment(SpringArmComp);

    // 创建并初始化 Interaction 组件
    InteractionComp = CreateDefaultSubobject<USInteractionComponent>("InteractionComp");

    // 创建并初始化 Attribute 组件
    AttributeComp = CreateDefaultSubobject<USAttributeComponent>("AttributeComp");

    // 创建并初始化 Action 组件
    ActionComp = CreateDefaultSubobject<USActionComponent>("ActionComp");

    // 设置角色的移动行为：使角色朝向移动方向，并禁用角色控制器的旋转
    GetCharacterMovement()->bOrientRotationToMovement = true; 	// 角色移动时自动朝向移动方向
    bUseControllerRotationYaw = false; 							// 禁用控制器旋转对角色偏航方向的控制

    // 在动画完成后禁用 Physics Asset 上的重叠查询，以提高性能
    GetMesh()->bUpdateOverlapsOnAnimationFinalize = false;

    // 允许角色的 Mesh 对象响应进入的投射物等
    GetMesh()->SetGenerateOverlapEvents(true);

    // 在胶囊碰撞体上禁用重叠查询，避免重叠查询的重复
    GetCapsuleComponent()->SetGenerateOverlapEvents(false);

    // 设置 HitFlash_CustomPrimitiveIndex 初始值为 0
    HitFlash_CustomPrimitiveIndex = 0;
}
```

### PostInitializeComponents

`PostInitializeComponents` 是 Unreal Engine 中一个重要的虚拟函数，它属于 `AActor` 类的一部分。这个函数通常用于在角色（或者其他继承自 `AActor` 的类）的组件初始化完成后执行自定义的初始化操作。

```C++
void ASCharacter::PostInitializeComponents()
{
	Super::PostInitializeComponents();					//调用父类的同名函数 Super::PostInitializeComponents()

	AttributeComp->OnHealthChanged.AddDynamic(this, &ASCharacter::OnHealthChanged);		//动态添加事件绑定
}
```

