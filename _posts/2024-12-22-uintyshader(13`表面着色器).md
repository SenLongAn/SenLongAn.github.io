---
title: UnityShade入门精要（13`表面着色器）
date: 2024-12-22 12:00:00 +0800
categories: [Unity_Shader]
tags: [shader]     # TAG names should always be lowercase
math: true
---
# 表面着色器

SurfaceShader是在顶点／片元着色器之上又添加了**一层抽象**，开发者更容易理解的方式，我们只需要简单的指定，要填充的颜色，法线，光照模型等，其他的全部交给unity自动完成（例如生成真正的顶点／片元着色器）

其中CG代码片段必须直接写在SubShader中，一个表面着色器中最重要的部分是两个结构体以及它的编译指令。

**两个结构体**是表面着色器中不同函数（shader）之间**信息传递**的桥梁， 而**编译指令**是我们和Unity沟通的重要手段（调用unity的内置API），最重要的作用可以指明着色器使用的函数和光照函数

## 编译指令

编译指令的一般格式如下：#pragma + 着色器名（表明用于什么着色器）+ 函数名 + 光照模型函数名 + （可选参数……）

指明光照函数（定义表面使用的光照模型），以便应用特定光照模型，Unity 内置了基于物理的光照模型函数Standard 和 StandardSpecular ，以及简单的非基于物理的光照模型函数Lambert和 BlinnPhong，也可以定义自己的光照函数

可选参数包括：自定义的修改函数（顶点(vertex:VertexFunction)（修改或传递顶点属性），颜色(:finalcolor:ColorFunction)（对最后的颜色进行修改）），阴影相关的代指令，透明度混合I透明度测试指令，光照指令，控制代码的生成……

## 两个结构体

表面函数（用于设置各种表面性质，如反射率、法线等）格式是固定的：

![1734932924762](/assets/img/blog/unityshader/表明函数格式.png)

SurfaceOutput、 SurfaceOutputStandard 和 SurfaceOutputStandardSpecular 都是 Unity 内置的结构体，它们需要配合不同的光照模型使用

Input 输入结构体包含了许多表面属性的数据来源

![1734932789619](/assets/img/blog/unityshader/Input数据.png)

输出结构体，如果使用了非基于物理的光照模型，我们通常会使用 SurfaceOutput 结构体，而如果使用了基于物理的光照模型Standard 或 StandardSpecular, 我们会分别使用 SurfaceOutputStandard 或 SurfaceOutputStandardSpecular 结构体

![1734932884562](/assets/img/blog/unityshader/表面输出结构体.png)

![1734932828273](/assets/img/blog/unityshader/输出结构体成员含义.png)

## 生成代码

根据表面着色器中的编译指令和自定义的函数，生成一个**包含了很多Pass的顶点／片元着色器**,可以在shader资源的 "Showgenerated code" 的按钮，查看生成的所有顶点／片元着色器代码

生成过程大致如下：表面着色器（黄色节点）->表面着色器流水线（黄色+灰色节点+指示线）->vert and frag（黑色方框）

![1734932964041](/assets/img/blog/unityshader/转换到vert和frag.png)

* 将源代码函数定义拷贝
* 定义编译指令，包含文件，宏等
* 生成顶点着色器
  * 根据需要选择（#ifdef 语句）（被所有自定义函数中使用的变量（包括了顶点位置、 纹理坐标、 法线方向、 逐顶点光照、光照纹理的采样坐标、视角方向、反射方向、阴影纹理坐标等）（包括Input结构体中被使用的成员））生成顶点着色器的输出一v2f_surf结构体，
  * 创建真正的顶点着色器，接受顶点数据（如appdata_full）
  * 顶点数据传入到可选的顶点修改函数（调用），修改后的顶点数据存入到v2f_surf结构体
  * 计算完成v2f_surf结构体传给片元着色器
* 生成片元着色器
  * 定义真正的片元着色器
  * 自定义的表面函数填充SurfaceOutput 结构体
  * 计算得到了光照衰减和世界空间下的法线方向，
  * 如果没有使用光照映射，SurfaceOutput 结构体传给自定义光照函数，计算得到初始的颜色值half4
  * 调用自定义的颜色修改函数，对输出颜色c进行最后的修改

## surfaceShader 的缺点

因为它是一种更高层的抽象，因此自由度较低，意味着任何在表面着色器中完成的事情，我们都可以在顶点／片元着色器中重现。但不幸的是，这句话反过来并不成立，并且表面着色器往往会对性能造成一定的影响

因此如果要使用大量光照，使用表面着色器，如果你需要处理的光源数目非常少，或者有很多自定义的渲染效果，那么使用顶点／片元着色器

![1735008313798](/assets/img/blog/unityshader/表面着色器.png)
