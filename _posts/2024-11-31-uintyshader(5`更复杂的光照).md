---
title: UnityShade入门精要（5·更复杂的光照）
date: 2024-11-31 12:00:00 +0800
categories: [Unity_Shader]
tags: [shader]     # TAG names should always be lowercase
math: true
---
# 渲染路径
渲染路径，决定了**光照信息**是如何在渲染管线中**传递**和**什么位置**进行**光照计算**的

unity主要有3 种渲染路径：前向渲染路径(ForwardRendering Path)、延迟渲染路径 (Deferred Rendering Path)，顶点照明渲染路径

大多数情况下， 一个项目只使用一种渲染路径，因此我们可以为整个项目设置渲染时的渲染路径。我们可以通过在Unity 的Edit-- Project Settings -- Player -- Other =Settings -- Rendering Path 中选择项目所需的渲染路径

但有时，我们希望可以使用多个渲染路径，可以在每个摄像机的rendering path设置

不同类型的渲染路径可能会包含多种标签设置，通过"LightMode"标签在pass通道指定pass**具体**的渲染路径（决定此pass具体进行**哪些光源**的光照计算），包含如下选项：

![1733062818899](/assets/img/blog/unityshader/渲染路径.png)

注意：
* 同时不要忘记预编译指令#pragma multi_compile_fwdbase / #pragmamulti_compile一fwdadd等,才可以保证光照变量可以被正确赋值
*  一个Base Pass仅会执行一次， 而一个AdditionalPass会根据影响该物体的其他逐像素光源的数目被多次调用，
* 对于一个物体来说， 环境光和自发光我们只希望计算一次即可，因此环境光和自发光是在BasePass中计算的， 而如果我们在AdditionalPass中计算这两种光照， 就会造成叠加多次环境光和自发光
* 开启和设置了混合模式。 这是因为，我们希望每个AdditionalPass可以与上一次的光照结果在帧缓存中进行叠加，
* Additional Pass中渲染的光源在默认情况下是没有阴影效果的， 中使用#pragma multi_compile_fwdadd _ fullshadows代替#pragma multi_ compile_ fwdadd编译指令，为点光源和聚光灯开启阴影效果，

###  前向渲染路径

前向渲染路径是传统的渲染方式，也是我们最常用的一种渲染路径,遍历每个片段，如果通过深度测试，就进行光照计算

假如场景中有N个物体，每个物体受M个光源的影响，那么要渲染整个场景一共需要N*M个Pass，可以看出，如果有大量逐像素光照， 那么需要执行的Pass数目也会很大

前向渲染光照计算位置可以在顶点也可以在像素阶段，取决于光照计算所处流水线位置

前向渲染的变量和函数：

![1733062832346](/assets/img/blog/unityshader/前向渲染可以使用的内置光照变量.png)

![1733062837762](/assets/img/blog/unityshader/前向渲染可以使用的内置光照函数.png)

### 前向渲染物体处理方式
按照什么方式处理光照：包括3种：逐顶点处理、逐像素处理，球谐函数 (Spherical Harmonics, SH) ，

取决于**光源类型**（平行，点，聚光）和**光源的渲染模式**（可以在unity editor光源对象的render mode设置）（该光源是否是重要的 (Important)）

Unity会根据场景中各个光源的设置以及这些光源对物体的影响程度（例如， 距离该物体的远近、 光源强度等）对这些光源进行一个重要度排序。
* 场景中最亮的平行光总是按逐像素处理的。
* 渲染模式被设置成NotImportant的光源，会按逐顶点或者SH处理。
* 渲染模式被设置成Important的光源，会按逐像素处理。
*  如果根据以上规则得到的逐像素光源数批小于Quality Setting中的逐像素光源数批(Pixel
Light Count), 会有更多的光源以逐像素的方式进行渲染

###  顶点照明渲染路径
是对硬件配罚要求最少、运算性能最高，但同时也是得到的效果最差的一种类型，它不支持那些逐像素才能得到的效果，例如阴影、法线映射、高精度的高光反射等

顶点照明渲染路径只是使用了逐顶点的方式来计算光照，通常在一个 Pass 中就可以完成对物体的渲染

![1733062845802](/assets/img/blog/unityshader/顶点照明渲染路径中可以使用的内置变量.png)

![1733062850034](/assets/img/blog/unityshader/顶点照明渲染路径中可以使用的内置函数.png)

###  延迟渲染路径
当场景中包含大量实时光源时，前向渲染的性能会急速下降,延迟渲染还会利用额外的G缓冲区, G 缓冲区存储了其他信息，例如该表面的法线、位置、用于光照计算的材质属性等。

延迟渲染使用的Pass数目通常就是两个，这跟场景中包含的光源数目是没有关系
的

在第一个Pass 中，我们不进行任何光照计算，而是仅仅计算哪些片元是可见的，当发现一个片元是可见的，我们就把它的相关信息存储到G缓冲区中,在第二个Pass 中，我们利用 G缓冲区的各个片元信息，进行真正的光照计算。

也就是仅执行像素次的光照计算，而非N*M次，效率依赖于屏幕分辨率，仅将最终通过深度测试后的片段执行光照计算，而非每次深度测试通过就执行

对于延迟渲染路径来说，它最适合在场景中光源数目很多、如果使用前向渲染会造成性能瓶颈的情况下使用，因为它有一些缺点：
* 不支持真正的抗锯齿 (anti-aliasing) 功能。
* 不能处理半透明物体。
* 显卡有一定要求。如果要使用延迟渲染的话，显卡必须支持 MRT (Multiple Render 
Targets)、 Shader Mode 3.0 及以上、 深度渲染纹理以及双面的模板缓冲

![1733062859898](/assets/img/blog/unityshader/延迟渲染路径中可以使用的内置变量.png)

###  pass
一个物体（model）应用一个Material（shader），即使用此shader的渲染管线渲染的model，其中Shader可以包含一个或多个Pass，一个Pass代表一次完整的渲染管线流程，即此model每帧被渲染pass次，
# 光源类型
我们之前一直模拟的是最简单的平行光，Unity 一共支持4种光源类型：平行光、点光源、聚光灯和面光源(arealigb t)（面光源仅在烘焙时才可发挥作用）

shader中最常使用的光源属性有光源的位置、方向 ,颜色、强度以及衰减

平行光：只有3个属性，方向，颜色，强度

点光源：位置，颜色，强度，衰减（球形）

聚光灯：位置，方向，颜色，强度，衰减
### 前向渲染路径处理不同光源类型实践
目标:物体受到平行光和一个点光源影响

由于使用了多个光源，我们需要多个pass，首先在'orwardBase中计算平行光，在ForwardAdd计算点光源，

还使用Blend命令开启和设置了混合模式，这是因为，我们希望Additional Pass计算得到的光照结果可以在帧缓存中与之前的光照结果进行叠加。如果没有使用Blend命令的话，Additional Pass会直接覆盖掉之前的光照结果

Ad中tionalPass中光照处理几乎和BasePass的处理方式是一样的，只是需要两点，去掉BasePass中环境光、自发光、逐顶点光照、SH光照的部分，并添加一些对不同光源类型的支持

由于Additional Pass处理的光源类型可能是平行光、点光源或是聚光灯，对于位置、方向和衰减这3个属性不同，因此需要根据光源类型分别计算

USING_DIRECTIONAL_LIGHT内置变量判断是否是平行光，defined (POINT)判断是否是点光源，defined (SPOT)判断是否是聚光

![1733069442725](/assets/img/blog/unityshader/point%20light.png)

### 光照衰减
有两种方法可以制造衰减效果：
* 光照衰减的纹理
```c++
float3 lightCoord = mul(unity_WorldToLight, float4(i.worldPos, 1)).xyz;
fixed atten = tex2D(_LightTexture0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL;
```
首先通过把顶点坐标变到光源空间，使用这个坐标的模的平方对衰减纹理进行采样，得到衰减值：
* 数学公式