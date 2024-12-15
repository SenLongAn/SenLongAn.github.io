---
title: UnityShade入门精要（11·噪声）
date: 2024-12-15 12:00:00 +0800
categories: [Unity_Shader]
tags: [shader]     # TAG names should always be lowercase
math: true
---
# 噪声

## 消融

噪声纹理＋透明度测试

当小于一定阈值时就会被clip剔除掉，烧焦的部分是颜色混合

禁用剔除，因为消融后可能呈现背面，在frag中，为了制造烧焦的效果，smoothstep当噪声 - 阈值 <= LineWidth时，越趋近0，越为烧焦的颜色

为了正确的获得阴影，在ShadowCaster模式下，使用阴影三剑客，并且在frag中clip剔除片段
## 水波
水波和之前实现的玻璃效果很像，需要反射（从cubemap采样），折射（GrabPass获得屏幕渲染的图像），不同的是，使用噪声纹理作为法线纹理，并且为了增加波光粼粼的效果，使用时间变量，采样噪声，对于反射折射的混合使用菲涅尔公式

## 全局雾效

和上一次实现的雾效几乎没什么区别，只不过使用了噪声让雾有变化些，采样得到的噪声 * fogDensity作为lerp颜色的混合因子

![1734281863917](/assets/img/blog/unityshader/消融，水.png)