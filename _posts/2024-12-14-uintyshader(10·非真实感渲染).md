---
title: UnityShade入门精要（10·非真实感渲染）
date: 2024-12-14 12:00:00 +0800
categories: [Unity_Shader]
tags: [shader]     # TAG names should always be lowercase
math: true
---
# 卡通风格

描边 + 纯色高光

有很多种方法实现描边：

我们之前用的方法3，这次使用方法2，在第一个pass先渲染背面，顶点沿着法线方向偏移viewPos + viewNormal * _Outline;，在frag使用轮廓线颜色，

第二个pass，在光照模型的镜面反射部分，依旧通过法线和半程h的dot点乘结果，通过step如果dot值>阈值（夹角<阈值）返回1，否则返回0，

但这样会导致高光锯齿，因此使用smoothstep达到平滑效果，当小于-w返回0，>w返回1，否则在0--1直接的插值，在根据此值lerp高光和非高光的颜色（也就是在高光边缘0--1的过渡）

# 素描风格

使用排调子渲染物体的暗部，越暗调子越密集，简化版的实现方式，准备不同的纹理，根据光照结果混合纹理颜色，

在vert中首先根据diff漫反射颜色值计算所有纹理的权重，在frag中将每个纹理采样的颜色和权重相乘，然后求和即可

![1734282092429](/assets/img/blog/unityshader/卡通风格和素描风格.png)