---
title: BreakOut_OpenGL下
date: 2024-11-14 12:00:00 +0800
categories: [OpenGL]
tags: [小游戏]     # TAG names should always be lowercase
math: true
---
# BreakOut_OpenGL下
## 粒子
在游戏中创造一些有趣的效果比如火焰，青烟，烟雾，魔法效果，炮火残渣等

粒子特效由众多粒子组成，每个粒子其实就是总是面向摄像机方向，且大部分区域是透明的纹理的小四边形（sprite），

粒子结构体有生命值，位置，速度，颜色属性，

粒子有额外的shader，使用vec2 offest位置控制粒子运动，它还包含uniform color颜色,让我们可以方便更改粒子颜色，粒子很小，我们会在.vs中直接*缩放10倍

通常会创建粒子生成器管理，它会不断生成新的粒子，旧的粒子会逐渐消亡，并且粒子的颜色可能会随着距离生成点越远颜色越暗（想象火焰）
* init初始化顶点数据，初始化粒子数组
* Update添加新的粒子，更新有效粒子的生命周期（减少），位置和颜色（a分量减少）
* Draw循环每个粒子，调用drawcall
* respawnParticle重置粒子属性，颜色位置随机，速度为父物体速度的0.1（如果速度一致就不会有拖尾效果）
* firstUnusedParticle首个无效粒子的索引（以便用新生成粒子更新它），一般情况即将无效的粒子总在上个无效粒子的右边，如果没有找到，在左边进行线性查找，再否则返回第一个粒子索引

![1731592913663](/assets/img/blog/breakout/bug.png)

这里的粒子life要设置短一些，否则会出现粒子不会断空情况，因为当vector满了，生命周期长，找不到无效的粒子，就只会更新0索引粒子(仅有一个粒子重置到玩家身上，其余仍以*0.1的速度移动)，因此出现断空

![1731592923347](/assets/img/blog/breakout/particle.png)

这样就可以正常跟随了
## 顶点数据 vs. uniform
顶点数据通常存放顶点特有（每个顶点都不同）属性（位置，纹理坐标，法线等）

uniform通常存放物体公有（每个顶点都相同）属性（MVP矩阵，光源位置，光源颜色等）

如果将公有属性放到顶点数据，会效率降低（每个顶点都会运行一次着色器）

如果将特有属性放到uniform，很难维护庞大的顶点数据
## 渲染位置
顶点数据

model矩阵

glm::lookat创建观察矩阵，模拟摄像机位置和旋转，根据传入摄像机位置，目标位置，上向量

glm::ortho（左，右，底，顶，近，远）创建投影矩阵，截取世界坐标系，设置视景体裁剪范围，超出范围的不渲染，最终转换为标准化设备坐标（-1 -- 1）
* 这里如果比如bottom大于top，y轴取反，图像延y方向颠倒绘制

![1731732118529](/assets/img/blog/breakout/ortho.png)

framebuffer大小，首次会映射到fb中，再由fb映射到视口

glviewport视口变换，用于关联窗口和裁剪区域，定义了最终渲染结果在窗口中的显示区域（会将裁剪范围转换为视口大小，呈现在窗口，如果视口小于窗口，剩余窗口未使用）

glfwCreateWindow显示在显示器的窗口大小

## postprocessing
打算制作三种特效：shake, confuse和chaos

* shake：**轻微晃动**场景并附加一个微小的**模糊**效果。
* confuse：**反转**场景中的颜色并**颠倒**x轴和y轴。
* chaos: 利用**边缘检测卷积核**创造有趣的视觉效果，并以**圆形旋转动画**的形式移动纹理图片，实现“混沌”特效。

这些后处理都需要framebuffer，步骤为：
* 绑定MSFBO（自定义多重采样的帧缓冲（离线渲染）），和往常一样渲染游戏，RBO附件作为（渲染缓冲附件（只读））渲染目标
* 将MSFBO内容Blit位块传输至FBO（自定义帧缓冲中（颜色缓冲附件））
* 解除绑定（绑定回默认的帧缓冲，它负责渲染到屏幕）
* 在post_Processor_shader中使用FBO的颜色缓冲纹理
* 渲染屏幕大小的四边形作为post_Processor_shader的输出

##### 卷积
卷积是一个强有力的数学工具，在CG/CV方面有非常不错的运用，能产生很多效果和输出

数学定义:

$$
F(n) = (f * g)(n) = \int_{-\infty}^{\infty} f(\gamma)g(n - \gamma)dr
$$
离散定义：
$$
F(n) = (f * g)(n) = \sum_{\gamma = 0}^{N} f(\gamma)g(n - \gamma)
$$
其中：
$$
n = \gamma + (n - \gamma)
$$

f (τ) 表示**被积**函数，而 g (n − τ) 表示**卷积核**函数。

f(x) 与 f ( τ )之间还存在着如下关系:
$$
f(x) = \int_{-\infty}^{\infty} f(\gamma)dr
$$

可以形象理解：

![1731634676014](/assets/img/blog/other/卷积.gif)
##### post_Processor_shader
shader：接受分支，决定纹理应用哪种后期处理效果

.vs：接受屏幕大小的quad的顶点位置（通常）和自定义帧缓冲的颜色缓冲纹理

.fs：改变颜色输出

抖动:根据时间（时间越长，移动越小）和强度值（强度越大，移动越大）影响quad渲染位置
```c++
float strength = 0.01;
gl_Position.x += cos(time * 10) * strength;        
gl_Position.y += cos(time * 15) * strength;   
```
颠倒贴图：xy纹理坐标取反
```c++
TexCoords = vec2(1.0 - texture.x, 1.0 - texture.y);
```
圆形旋转动画
```c++
float strength = 0.3;
vec2 pos = vec2(texture.x + sin(time) * strength, texture.y + cos(time) * strength);        
TexCoords = pos;
```
根据offests(被积偏移量)采样颜色
```c++
for(int i = 0; i < 9; i++)
    sample[i] = vec3(texture(scene, TexCoords.st + offsets[i]));
```
反转颜色
```c++
color = vec4(1.0 - texture(scene, TexCoords).rgb, 1.0);
```
边缘检测卷积核edge_kernel
```c++
for(int i = 0; i < 9; i++)
    color += vec4(sample[i] * edge_kernel[i], 0.0f);
    color.a = 1.0f;
```
模糊卷积核blur_kernel
```c++
for(int i = 0; i < 9; i++)
    color += vec4(sample[i] * blur_kernel[i], 0.0f);
    color.a = 1.0f;
```
##### PostProcessor
后期处理管理器

* 构造函数：初始化，两个帧缓冲，quad顶点数据，卷积核以及偏移量
* BeginRender绑定MSFBO（使用额外的帧缓冲，多重采样目的获得更好的效果，通过glRenderbufferStorageMultisample（））
* EndRender位块传输到FBO，解除绑定
* Render绘制（bind + drawcall）
##### 修改Game类
在渲染中执行正确的bind

我们只想允许晃动效果持续一小段时间，在碰撞触发时更改uniform变量为true
## 设计开发一些Gameplay/优化模块
关卡循环：4个关卡循环

窗口大小调整：需要回调帧缓冲，视口两个

砖块根据颜色区分生命值（1，2，3，4），只有当生命值为0才会销毁，否则反弹，并将a分量降低为 当前生命 / 总生命
## 道具
本节加入道具（Gameplay）模块，任何一个砖块在被销毁时都有一定几率产生一个道具块，这样的道具快会缓慢降落，而且当它与玩家挡板发生接触时，产生对应效果，否则如果超出屏幕销毁

上一节制作了3种特效，通过简单修改unifrom变量就可以启用，我们设置后两种通过道具触发  

会有以下6种道具（4种增益道具与2种负面道具）：

* Speed: 增加小球20%的速度
* Sticky: 当小球与玩家挡板接触时，小球会保持粘在挡板上的状态直到再次按下空格键，这可以让玩家在释放小球前找到更合适的位置
* Pass-Through: 非实心砖块的碰撞处理被禁用，使小球可以穿过并摧毁多个砖块
* Pad-Size-Increase: 增加玩家挡板50像素的宽度
* Confuse: 短时间内激活confuse后期特效，迷惑玩家
* Chaos: 短时间内激活chaos后期特效，使玩家迷失方向
##### Powerups
Powerups道具类继承游戏对象，每个道具以字符串的形式定义它的类型
它有持续时间（0表示无限），类型，是否激活的额外属性，定义vector存储上述6种道具，以及一些函数：
* ShouldSpawn给定int，通过随机值，返回bool（每种道具有1/int概率为真）
* SpawnPowerUps根据ShouldSpawn随机生成一个/多个道具，存放在vector（每个道具最多一次）
* isOtherPowerUpActive给定类型，获取是否是激活的
* UpdatePowerUps更新所有当前被激活的道具的属性（位置，持续时间），当Duration结束，会根据道具类型取消当前效果
* ActivatePowerUp会根据道具类型激活道具效果

##### 修改Game类
当销毁砖块，生成道具，与玩家碰撞使用道具，否则销毁
Sticky和Pass-through道具稍微改变了一些原有的游戏逻辑，因此会修改Ball类（增加了两个属性） ……
![1731681600324](/assets/img/blog/breakout/Post.png)
## 音乐音效
IrrKlang是一个可以播放WAV，MP3，OGG和FLAC文件的高级二维和三维（Windows，Mac OS X，Linux）声音引擎和音频库。它还有一些可以自由调整的音频效果，如混响、延迟和失真。
使用非常简单，只需要导入库,初始化声音引擎，并通过play播放音乐，bool控制是否循环
```c++
#include <irrklang/irrKlang.h>
using namespace irrklang;
ISoundEngine *SoundEngine = createIrrKlangDevice();
SoundEngine->play2D("…….mp3", GL_TRUE);
```
## 文本
添加TextRenderer:文本渲染类，Text shaders文本着色器
* 构造：
* Load指定字体文件和字体大小，会加载128个ASCII的字符，将结果（字形纹理（库生成的），大小，垂直偏移，水平偏移）存储在map Characters中
* RenderText,渲染字形，接受字符串，指定位置，大小，颜色，循环每个字符渲染，这里由于orthor中设置（y值取值从顶部到底部递增），需要改变计算垂直方向偏移的方法

FT_Library ，FT_Init_FreeType初始化字体库
FT_Face ，FT_New_Face字体（字符的样式）文件
FT_Set_Pixel_Sizes定义字体大小
FT_Load_Char将其中一个字形（单个字符的具体形状）设置为激活字形

##### 增加gameplay
√正常情况，游戏关卡循环
√左上显示当前关卡，销毁的砖块计数，死亡次数
√按下tab调出重置游戏菜单：暂停游戏：停止update逻辑（游戏对象属性），渲染正常
√按键w s切换开始关卡，
√enter重置关卡，会重置所有数据（窗口，音乐）
√如果未选择再次按下q，不会发生任何
√字体颜色

-------------------------------------OpenGL 小游戏实战 完结--------------------------------------------

## 项目演示
[bilibili演示](https://www.bilibili.com/video/BV1tZUaYNE7B/?spm_id_from=333.999.0.0&vd_source=399848dcdb86de5af61de51a6dd0d33b)

