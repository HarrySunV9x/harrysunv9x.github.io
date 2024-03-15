---
title: 摄像机组件Cinemachine
date: 2024-02-04
---

# 摄像机组件Cinemachine

在Unity中，传统的摄像机组件可以通过编写脚本进行控制。Unity有一个先进的Unity摄像机系统：Cinemachine——它是一个免费的Unity插件，它极大地简化了动态和电影级相机效果的创建过程，无需大量编程即可实现。

## 安装与创建

**安装：**

1. 在Unity编辑器中，点击`Window`菜单。
2. 选择`Package Manager`。
3. Packages: xxx，选择`Unity Registry`。
4. 搜索`Cinemachine`。
5. 点击`Install`进行安装。

**创建：**

1. 在Hierarchy视图的空白区域或者导航栏的GameObject项目选择Cinemachine。
2. 选择需要的相机类型。

## 组件介绍

1. **2D Camera**
   - 专门为2D游戏设计的相机，可以处理平面跟踪和平滑移动。
2. **Blend List Camera**
   - 允许创建一个相机列表，这些相机可以根据指定的顺序和时间混合它们的镜头。这对于创建复杂的镜头切换序列非常有用。
3. **ClearShot Camera**
   - 这是一个智能相机，可以在多个相机之间选择最佳的镜头。它会自动避开遮挡，并寻找最清晰的视角来拍摄目标。
4. **Dolly Camera with Track**
   - 该相机沿着一个预先设定的轨道移动，类似于现实世界中的摄影机轨道系统。它用于创建平滑的、有轨道的镜头移动效果。
5. **Dolly Track with Cart**
   - 类似于Dolly Camera with Track，但它提供了一个“购物车”（Cart），可以在轨道上前后移动，以达到更加精细的控制。
6. **FreeLook Camera**
   - 提供了一个围绕目标的三个轴的自由视角，玩家可以自由旋转相机来观察周围环境。
7. **Mixing Camera**
   - 允许混合两个或更多相机的属性，如位置和旋转，这样可以在不同相机视角之间创建动态过渡。
8. **State-Driven Camera**
   - 通过游戏对象的状态来驱动相机的选择和行为。这通常与动画状态机集成，根据角色或游戏的状态改变相机。
9. **Target Group Camera**
   - 这个相机可以同时关注多个目标，并根据配置的规则在它们之间平衡视角和焦点。
10. **Virtual Camera**
    - 这是Cinemachine的基础组件，代表一个单独的相机镜头。它可以被配置为跟踪一个目标，或者实现特定的相机行为，如移动、旋转等。

固定视角的常用Virtual Camera，第三人称常用FreeLook Camera，灵活取用。

## FreeLook Camera

通过三组相机实现的自由视角，默认就可以通过鼠标移动视角。

Hierarchy视图中选择FreeLook Camera对象，Sense界面可以看到一个桶形的范围指示器，它代表了相机的可移动范围。将中间的圆放在肩膀位置，头顶和脚底的圆半径为0.001（防止反转），就可以实现类原神的第三人称视角。