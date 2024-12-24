---
title: UnityShade入门精要（6·cubmap，环境映射，MRT，程序化纹理）
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

### cubmap实例：

我们需要根据物体位置不同动态生成cubmap，从而根据这个cubmap采样，因为不同的物体位置，对应的环境也不同，是否有遮挡，离谁更近，必须从物体的位置生成环境，才能得到正确的反射颜色

比如如图所示，如果新的物体位置仍然从第一个物体生成的环境采样，将不会有挡板反射，得到不正确的采样

![1735024863921](/assets/img/blog/unityshader/不同的cubmap.png)

写一个继承自: ScriptableWizard的类（编辑器扩展）（这样做是为了在运行游戏前创建生成），

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

菲涅耳反射 (Fresnelreflection) 根据视角方向控制反射光线占（散射+吸收）总光线的比率（当视角和法线越趋近于水平，反射光线占比越多）,可以在边界处模拟反射光强和折射光强／漫反射光强之间的变化(比如车漆、水面等材质来模拟更加真实的反射效果)

如何计算菲涅耳反射呢？

其中一个著名的近似公式就是Schlick 菲涅耳近似等式：

F schick(V ,N)=F0 + (l-F0)(1-v·n)^5,F0是一个反射系数，用于控制菲涅耳反射的强度，

另一个应用比较广泛的等式是Empricial菲涅耳近似等式:

F empricial(v,n)=max(O,min(l,bias+scale X (1- v·n)^power)),其中， bias、 scale 和power 是控制项

使用系数：lerp(diffuse, reflection, saturate(fresnel))，系数越大，反射能力（环境映射能力）越强，占混合比例越大

### MRT

现代的 GPU 允许我们渲染到渲染目标纹理 (Render Target Texture, RTT),而非直接的颜色缓冲中（直接呈现在屏幕），并且我们也可以同时渲染到多个渲染目标纹理中，即多重渲染目标(Multiple Render Target, MRT)（比如延迟渲染）

Unity 为渲染目标纹理定义了一种专门的纹理类型一—渲染纹理 (RenderTexture)

在Unity中使用渲染纹理通常有两种方式：

一种方式是在Project 目录下创建一个渲染纹理，然后把某个摄像机的渲染目标设置成该渲染纹理，这样一来该**摄像机**的渲染结果就会实时更新到渲染纹理中，而不会显示在屏幕上

另一种方式是在屏幕后处理时使用 GrabPass 通道 命令或 OnRenderimage 函数 来获取当前**屏幕**图像，Unity 会把这个屏幕图像放到一张和屏幕分辨率等同的渲染纹理中

### 镜子效果

创建一个渲染纹理Create->Render Texture，创建一个新的摄像机，Render Texture作为摄像机的TargetTexture，这样就将相机观察到的场景渲染到了目标纹理中

镜子的原理很简单，渲染纹理在水平方向上翻转后直接显示到物体上即可

### 玻璃效果

通常会使用 GrabPass 来实现诸如玻璃等透明材质的模拟，如果单纯使用blend，绘制的不像是玻璃（因为折射会发生观察目标位置改变），如果单纯使用上述的环境映射_折射，只能从cubmap采样，模拟充满介质（如水）的物体，不能实现混合颜色

因此想要实现玻璃，需要混合+折射，通过GrabPass来实现,获取玻璃后面的屏幕图像(blend效果),对屏幕纹理坐标偏移后， 再对屏幕图像进行采样来模拟近似的(折射效果)

不要忘了ransparent的渲染队列为透明队列，GrabPass { "_RefractionTex" }传入的参数即为渲染的屏幕图像（因为未绘制当前本身，是先绘制的那部分纹理的图像，即物体后面的图像）

可选的法线贴图（和之前一样，可以让玻璃体现凹凸感），_RefractAmount参数作为 玻璃反射（本身颜色）和折射（缓冲的颜色）混合因子，反射部分依旧从cubmap采样

### 渲染纹理 vs. GrabPass

GrabPass在实现上相对简单，只需要在shader中写几行代码就可以实现抓取屏幕的目的，但是，渲染纹理的效率往往要好于GrabPass

### 程序纹理

程序纹理 (Procedural Texture)使用一些特定的算法来创建个性化图案或非常真实的自然元素， 例如木头、石子等

波点纹理实例：

shader还是使用之前的使用基础texture的shader，这次使用一个新的cs脚本程序化生成纹理

❌error CS0246: The type or namespace name ‘SetProperty‘ could not be found：

由于缺少一个用于Unity材质属性编辑的开源插件，通过访问GitHub并下载SetProperty插件

[ExecuteInEditMode]可以让脚本在editor模式下执行

cs中利用插件的SetProperty将几个属性暴露给editor面板

#region 和#endregion 仅仅是为了组织代码，并没有其他作用

在程序化生成纹理的函数中，遍历每个纹理像素，

在内部每个像素又遍历9个圆形，依次计算出圆心的位置，再求像素和每个圆心的距离，距离越大越趋近于背景颜色，否则越趋近园的指定色（每次颜色都和之前求得的颜色根据距离lerp插值），在循环9次后，对像素SetPixel设置颜色，

在所有像素遍历完成.Apply()强制把像素值写入纹理中，并返回该程序纹理

### 程序材质

专门使用程序纹理的材质，叫做程序材质，并不是在 Unity 中创建的，而是使用了一个名为 SubstanceDesigner 的软件在 Unity 外部生成的。

![1733588433778](/assets/img/blog/unityshader/高级贴图.png)
