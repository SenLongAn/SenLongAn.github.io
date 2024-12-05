---
title: UnityShade入门精要（6·高级纹理）
date: 2024-12-4 12:00:00 +0800
categories: [Unity_Shader]
tags: [shader]     # TAG names should always be lowercase
math: true
---
# 立方体纹理
如果要绘制天空盒需要使用立方体纹理（包含6张纹理的统称），映射到一个立方体上，立方体包裹住摄像机

从天空盒环境映射到物体上，从而让物体投影天空盒纹理，如何对立方体贴图采样？

需要3维纹理坐标，方向矢量从立方体中心出发，和立方体求交点即求纹理坐标

### 天空盒实例：

首先在unity创建天空盒非常简单，只需要新建材质，选择 Unity 自带的Skybox/6 Sided着色器，该材质需要 6 张纹理（我们需要把这6张纹理的WrapMode设翌为Clamp, 以防止在接缝处出现不匹配的现象）

为了应用skybox材质，在Window->Lighting 菜单中，把 SlcyboxMat 赋给 Slcybox 选项，并且摄像机的 Camera 组件中的Clear Flags 被设胃为 Skybox

如果我们希望某些摄像机可以使用不同的天空盒子，可以为摄像机添加Skybox组件来覆盖掉之前的设置

### 创建cubmap实例：

我们需要根据物体位置不同动态生成cubmap，从而根据这个cubmap采样，写一个继承自: ScriptableWizard的类（编辑器扩展）（这样做是为了在运行游戏前创建生成），

脚本：创建临时camera，移动到指定位置，RenderToCubemap函数，把从当前位置观察到的图像渲染到用户指定的立方体纹理cubemap 中

创建一个空的 GameObject 对象，作为生成cubmap的位置

新建一个用于存储的立方体纹理Create 一 Legacy 一Cubemap，为了让脚本可以顺利将图像渲染到该立方体纹理中，我们需要在它的面板中勾选Readable选项。

在untiy editor指定路径下打开编辑器扩展面板，拖放刚才创建的两个obj，点击建立的按钮，即可完成渲染到cubmap

### 反射实例
我们只需要、 通过入射光线的方向和表面法线方向来reflec计算反射方向，再利用反射方向对立方体纹理采样即可

通过lerp函数可以根据参数值控制颜色混合
### 折射实例
斯涅尔定律 (Snell's Law) 来计算反射角，正确的模拟需要refract计算两次折射，进入内部，从内部射出，但其实在实时渲染中我们通常仅模拟第一次折射。

反射，折射

![1733416461950](/assets/img/blog/unityshader/cubmap.png)
### 菲涅耳反射实例