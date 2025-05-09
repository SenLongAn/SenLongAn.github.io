---
title: UnityShade入门精要（7·动画效果）
date: 2024-12-7 12:00:00 +0800
categories: [Unity_Shader]
tags: [shader]     # TAG names should always be lowercase
math: true
---
# 动画效果
要实现动画效果往往和时间变量有关，内置的时间变量：

![1733588337283](/assets/img/blog/unityshader/内置时间.png)

### 序列帧动画
依次播放一系列关键帧图像，当切换间隔时间达到一定数值时， 看起来就是一个连续的动画

我们需要准备一张包含关键帧的图像，它们的大小相同，

由千序列帧图像通常是透明纹理，我们应该向透明度章节一样，使用透明队列，禁用深度写入，开启blend混合模式

内置宏 TRANSFORM_TEX这里回顾一下，它进行下述过程，.xy对顶点纹理坐标进行缩放，然后再使用偏移属性.zw 对结果进行偏移

我们不再是采样整张的纹理贴合到物体上，而是从上到下，从左到右（横着一行行读，具体看序列帧图片的布局）采样局部的纹理作为物体的纹理

为了实现这个效果，首先获取当前的time，CG的floor函数对结果值取整来得到整数时间 time，

使用 time除以_Horizontal.Amount的结果值的商来作为当前对应的行索引，除法结果的余数则是列索引，这样就获得当前时间下对应的纹理索引

让当前纹理坐标+偏移（纹理坐标范围是0--1）：i.uv + half2(column, -row);（列对应x坐标，行对应y坐标），再/Amount，即可获得对应的区域纹理坐标

### 纹理动画
纹理动画通常会使用多个层 (layers) 模拟无限滚动，因为每个层模拟了不同的深度，近处的移动快，远处的移动慢，具有不同的移动速度

移动本质就是将纹理坐标根据时间偏移，frac(float2(_ScrollX, 0.0) * _Time.y);通过frac取小数部分（即0--1）达到循环效果，其中_ScrollX速度值越大，越容易进一个整数位（纹理循环一次）
### 顶点动画
DisableBatching禁用批处理操作，因为批处理会合并所有相关的模型，而这些模型各自的模型空间就会丢失

波浪效果：通过sin函数计算顶点位置

广告牌技术：让quad看起来好像总是面对着摄像机 ，本质就是构建旋转矩阵，需要3个基向量：表面法线 (normal)、指向上的方向 (up) 以及指向右的方向 (right)，还需要指定一个描点 (anchorlocation), 这个铀点在旋转过程中是固定不变的，

为了计算新的正交基（即新的quad方向对应的坐标），首先确定法线方向，即轴点和视线方向，确定up方向，但是这时通常不会和法线垂直，需要先叉积计算right方向，再right和法线叉积计算旋转后的up方向

如何将quad的顶点变换到新的方向呢？每个分量分别和正交基分量相乘，即得到新的旋转位置

unity shader中的MVP都是通过在unity editor下调节的，如果想要在运行时改变，需要这样计算

### 顶点动画的阴影
使用内置的 Diffuse 等包含的阴影 Pass 来渲染，就得不到正确的阴影效果,ShadowCasterPass 中并没有进行相关的顶点动画，

解决方式就是自定义ShadowCaster，之前渲染波浪的pass是不变的，新增pass，使用offest后的顶点位置对阴影贴图采样，

![1733588382377](/assets/img/blog/unityshader/动画.png)