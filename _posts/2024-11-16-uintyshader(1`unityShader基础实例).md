---
title: UnityShade入门精要（1·unityShader基础和Debug）
date: 2024-11-16 12:00:00 +0800
categories: [Unity_Shader]
tags: [shader]     # TAG names should always be lowercase
math: true
---
之前理论章节先略过，有时间再看，从第5章实战部分开始

渲染管线简化流程：data->CPU->GPU->演示，的过程，其中我们可以更改其中的shader可编程部分，完成不同的最终演示效果

在引擎中应用shader非常方便，其它的都是unity editor中的可视化的图形，仅需要将写好的shader应用到物体材质上，这个物体的渲染就会用这个着色器，那么根据shader的不同，呈现的最终效果也不同

# 为obj应用shader

新建unity工程

在assest新建shader：create->shader->……有如下分类：

* Standard Surface Shader标准**表面着色器**，可以实现各种**物品的效果**，比如石头、木材、玻璃、塑料和金属等等。
* Unlit Shader它是最**简单的着色器**，由最基础的 **Vertex Shader 和 Fragment Shader**组成
* Image Effect Shader主要针对实现各种屏幕**后期处理着色器**，一般像是泛光、调色、景深、模糊等基于最终整个屏幕画面而进行再次处理的就是后处理，
* Compute Shader**计算着色器**，独立于常规渲染管线之外的，它可以直接将GPU作为并行处理器加以利用，使GPU具有运算能力。一般会在需要**大量并行计算**的时候使用。
* Ray Tracing Shader**光线追踪着色器**，光线追踪可以轻松模拟各种**光学效果**，如反射、折射、散射、色散等。但由于在进行求交计算时需要知道整个场景的信息，它的计算成本也是非常高的。

为了更原始，可以在window->lighting->environment把dufault天空盒去掉

创建一个shader，简单效果通常创建unlit shader，

新建材质，将shader给它 （在材质的inspector面板上面有选择shader，搜索你的shader名称），

新建球体，把材质给它（mesh renderer组件中）

# 编写shader_CG

unity使用ShaderLab对shader进行了更高一层的封装

### 基本结构

首先看一下它的基础结构：

shader的路径“……/……”

Properties定义可以在材质编辑器中调整的属性

SubShader

* LOD细节层次和渲染标签Tags
* pass 渲染通道，
  * CGPROGRAM和ENDCG包裹了实际用 Cg/HLSL/GLSL 编写的顶点和像素着色器代码（本教程使用CG，类似于C语言）

Fallback指定了如果当前着色器无法正常工作时使用的备用着色器

## 看一下最简单的实例

将绘制白色物体

```c++
Shader "Unlit/SimpleShader"
{
    SubShader
    {
        Pass
        {
            CGPROGRAM
            //哪个函数包含了顶点着色器的代码，哪个函数包含了片元着色器的代码。
            #pragma vertex vert
            #pragma fragment frag
            //逐顶点：接受顶点位置，返回裁剪空间的位置，
            float4 vert(float4 v: POSITION) : SV_POSITION {
                return mul (UNITY_MATRIX_MVP, v);
            }
            //逐像素
            fixed4 frag() : SV_Target{
                return fixed4(1.0, 1.0, 1.0, 1.0); 
            }
            ENDCG
        }
    }
}
```

### 如果我们想要得到其他的模型的顶点数据怎么办呢？

输入**参数和返回值**不再是一个简单的数据类型，而是一个**结构体**,

Unity 会根据这些语义来填充这个结构体,现在这些成员都是有数据的、

Unity 支持的语义有（由MeshRender组件提供）： POSITION, TANGENT, NORMAL, TEXCOORDO, TEXCOORDJ, TEXCOORD2, TEXCOORD3, COLOR 等

```c++
struct a2v {//application to vertex
  float4 vertex : POSITION;
  float3 normal : NORMAL;
  float4 texcoord : TEXCOORD0;
};
```

顶点着色器和片元着色器之间如何通信？

因为顶点数据只能通过vs传给fs（不考虑uniform）

为此需要使用一个新的结构体v2f来定义顶点着色器的输出（要指定语义）

让vert函数返回v2f，frag接受v2f参数
![1732980973590](/assets/img/blog/unityshader/球体.png)

### 属性

属性可以理解为其他图形API的**uniform**变量

材质和Unity Shader之间的紧密联系。材质提供给我们一个可以方便地调节Unity Shader中参数的方式， 通过这些参数， 我们可以随时调整材质的效果。而这些参数就需要写在Properties语义块中

我们想要在材质面板显示一个颜色拾取器， 从而可以直接控制模型在屏幕上显示的颜色

```c++
Properties {
    _Color ("Color Tint", Color) = (1, 1, 1, 1)
}
```

在这里定义了一个属性_Color， 它的类型是Color, 初始值是(1.0,1.0, 1.0, 1.0), 对应白色

为了在CG代码中可以访问它， 我们还需要在CG代码片段中提前定义一个新的变量， 这个变量的名称和类型必须与Properties语义块中的属性定义相匹配

![1731898517840](/assets/img/blog/unityshader/Shaderlab属性类型和CG变量类型.png)

### uniform关键字

uniform 关键词是 CG 中修饰变量和参数的一种修饰词，它仅仅用于提供一些关于该变量的初
始值是如何指定和存储的相关信息（这和其他一些图像编程接口中的 uniform 关键词的作用不太一样）。在Unity Shader 中， uniform 关键词是可以省略的。

# uinty提供的内置文件和变量

为了方便开发者的编码过程， Unity 提供了很多内置文件,这些文件包含了很多提前定
义的函数、变量和宏等。
以使用#include 指令把这些文件(后缀是.cginc)包含进来，这样我们就可以使用 Unity 为我们提供的一些非常有用的变量和帮助函数
我们可以从Unity 的应用程序中直接找到CGincludes 文件夹，它包含了所有内置着色器包含文件
![1731911513981](/assets/img/blog/unityshader/常用包含文件.png)
![1731911522599](/assets/img/blog/unityshader/UnityCG.cginc中一些常用的结构体.png)
![1731911527773](/assets/img/blog/unityshader/UnityCG.cginc中一些常用的帮助函数.png)

# CG/HLSL语义

CG/HLSL提供的语义(semantics)， 语义实际上就是一个赋给Shader输入和输出的额外**字符串**（表达了这个参数的含义）。 通俗地讲， 这些语义可以让Shader知道从哪里读取数据， 并把数据输出到哪里，

如果没有这些语义来限定输入和输出参数的话，渲染器就完全不知道用户的输入输出是什么，因此就会得到错误的效果

是系统数值语义(system-valuesemantics)以sv开头的， sv代表的含义就是系统数值(system-value)， 这些语义在渲染流水线中有特殊的含义，是不可以随便赋值的， 因为流水线需要使用它们来完成特定的目的，
![1732067510752](/assets/img/blog/unityshader/从应用阶段传递模型数据给顶点着色器时Unity支持的常用语义.png)
![1732067517184](/assets/img/blog/unityshader/从顶点着色器传递数据给片元着色器时Unity使用的常用语义.png)
![1732067525784](/assets/img/blog/unityshader/片元着色器输出时Unity支持的常用语义.png)

# Debug

普通程序的bug通过传统debug方式解决

opengl和vualkan包含一些内置的调试工具，可以方便我们查看哪里整个渲染流程的语法/遗漏错误

shader中的语法错误，opengl会自动告诉你

还需要手动数据可视化调试，即将数据映射到0--1的范围，作为颜色输出，从而debug模型顶点数据是否正确

//
调试工具脚本：颜色拾取器
在GUI创建box（显示纹理和背景）和label文本（显示rgba）
创建BoxCollider以接收鼠标事件
当鼠标点击OnMouseDown()执行，获取鼠标位置
OnPostRender()每帧执行，渲染屏幕大小纹理，根据鼠标位置从纹理获取颜色
为box应用方形纹理（应用颜色）
