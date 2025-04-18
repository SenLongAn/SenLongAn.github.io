---
title: BreakOut_OpenGL上
date: 2024-11-11 12:00:00 +0800
categories: [OpenGL]
tags: [小游戏]     # TAG names should always be lowercase
math: true
---
# BreakOut_OpenGL上

用openGL.C++制作一个经典2D游戏——Breakout逃脱

#### 游戏机制：

游戏要求玩家通过操控一个挡板在屏幕范围内左右移动

还有一个小球，在屏幕上运动，每次和(屏幕边框，砖块，挡板)碰撞都会使球根据碰撞位置改变运动方向

当球运动到屏幕底部边界的时候，游戏结束（失败）

当破坏所有的砖块，游戏结束（胜利）

## 游戏框架

对于项目涉及的所有模块，会进行下面的设计，以便解耦，更易扩展和组合，提高复用性等

#### Game类：

负责渲染相关和游戏代码，

从窗口中解耦，以便方便把游戏迁移到不同的窗口库中

将渲染和逻辑控制分开

init初始化 `<br/>`
update循环 `<br/>`
render渲染 `<br/>`
input输入 `<br/>`
初次之外还有一个游戏流程的enum ，保存GameState游戏状态

#### 工具类：

我们会频繁重用一些内容
Shader着色器类：生成着色器程序Shader Program，快速设置uniform值glUniform***
Texture2D纹理类：Generate生成，Bind绑定2D纹理

#### 资源管理器：

资源管理器，成员都是static静态函数
接受（字节，字符串，文件路径），调用工具类加载资源，和通过map保存已加载的资源

#### 项目文件

program主程序main
window窗口类

Game游戏层类

shader着色器类（工具类）
texture纹理类（工具类）
debug调试器（工具类）
header第三方库引用（工具类）
resource_manager资源管理器（工具类）

game_level游戏关卡（游戏对象vec）
game_object(游戏对象)
ball_object球体（游戏对象，继承game_object）
power_up道具类（游戏对象，game_object）

sprite_renderer精灵渲染器（渲染）
particle_generator粒子生成器（渲染）
text_renderer文本渲染器（渲染）
post_processor后期处理

## 创建窗口

LINK2019❌：程序找不到库

* 是否在property->link->input添加了glfw3dll.lib，opengl32.lib，glew32.lib
* 是否添加#define GLEW_STATIC
* 是否有glewInit();

重复包含❌：OpenGL header already included, remove this include, glad already provides it

* #include <GL/glew.h> 和 #include <glad/glad.h> 只能选择其中之一使用

找不到 **.dll❌：

* 将dll直接拷贝到和应用程序exe同一文件夹中

unresolved❌：
unresolved external symbol stbi_load referenced in function “unsigned int __cdecl loadTexture

* 在#include "stb_image.h"前加上，#define STB_IMAGE_IMPLEMENTATION

//
本节目标是在program(创建窗口，初始化glew，设置渲染配置，调用game的逻辑，主while，KeyCallback)

以及所有和4个辅助文件逻辑关联起来

Game，shader，texture，resource_manager

## render sprite

我们希望在窗口中渲染精灵（纹理的四边形），并可以方便控制它的纹理，颜色，大小，旋转，位置等

要渲染obj需要走渲染流程，需要顶点数据（包括VAO等），sprite着色器

顶点着色器需要接受顶点数据，经过MVP变换，片段着色器接受sampler2D纹理，通过纹理坐标查找颜色（和之前没有区别）

定义一个sprite_renderer渲染器类（从游戏代码抽象出渲染代码）

它有顶点数据（坐标，纹理坐标）通过initRenderData函数配置VAO

通过DrawSprite函数，绑定shader，纹理，uniform，VAO，最后调用DrawCall渲染

```c++
 model = glm::translate(model, glm::vec3(position, 0.0f));  // first translate (transformations are: scale happens first, then rotation, and then final translation happens; reversed order)

 model = glm::translate(model, glm::vec3(0.5f * size.x, 0.5f * size.y, 0.0f)); // move origin of rotation to center of quad
 model = glm::rotate(model, glm::radians(rotate), glm::vec3(0.0f, 0.0f, 1.0f)); // then rotate
 model = glm::translate(model, glm::vec3(-0.5f * size.x, -0.5f * size.y, 0.0f)); // move origin back

 model = glm::scale(model, glm::vec3(size, 1.0f)); // last scale
```

你可能会疑惑额外的两次translate是干什么的？

我们顶点数据定义quad的左上角在0，0位置，因此它会以左上角旋转，而非quad中心点，因此只需要首先先把它移到屏幕中心（局部坐标），应用旋转，再移回原位

这里要注意，变换矩阵顺序是缩放，旋转，位移，因为矩阵乘法是从右向左执行的，所以我们变换的矩阵顺序是相反的：移动，旋转，缩放

在顶点着色器中，先应用缩放，然后移动到中心点旋转再移回原位，最后进行变换到世界空间位置
//
现在通过在Game中调用sprite_renderer提供的接口，我们可以非常方便的渲染一个/多个sprite_quad，也非常方便应用不同属性

unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);Exception thrown at 0x0000000000000000 in BreakOut.exe: 0xC0000005: Access violation executing location 0x0000000000000000.❌：

调用glCreateShader必须初始化glfw,以及glew，可以检查是否有初始化错误，或者调用放在了初始化前

测试：

![1731466746442](/assets/img/blog/breakout/sprite.png)

## level

我们想要渲染许多彩色砖块以及挡板组成的完整关卡，可以灵活的调整行或列，砖块类型，砖块布局

创建gamelevel类，负责关卡创建

使用文本文件建立布局：0（无砖块），1（不可被摧毁），>1（2，3，4，5）（可以被摧毁，不同数字代表不同颜色）

* Load函数负责获取文本内容，将数字加载到2维数组中，调用init函数
* init函数，会遍历每个数字，并根据值生成gameobject类对象，并添加到Bricks数组中
* Draw负责遍历每个未销毁的砖块渲染
* IsCompleted验证是否所有的可破坏砖块均被摧毁

创建一个GameObject游戏对象

* 包括位置，颜色，是否可被破坏，是否被销毁，纹理等属性
* 调用draw即调用sprite渲染

因为希望有多个关卡，因此在game中创建gamelevel数组保存关卡，会根据GLuint值告知当前的关卡索引（我们有4个关卡）

创建一些文本文档（关卡）

Game::Init中初始化关卡，并保存到gamelevel数组（不要删除之前的shader部分）

并在render中渲染背景和关卡

加入挡板,创建一个gameobjct，在render中绘制，然后在ProcessInput通过GLboolean接受用户输入，控制对象的属性更新

![1731478726248](/assets/img/blog/breakout/player.png)

## 球

关卡中还差球体，本节既要渲染，也要球体首先能够与屏幕边缘交互

我们的砖块和挡板都是gameobject游戏对象，球同样也是游戏对象，并具有额外的（半径，是否被stuck固定）属性，除了渲染外，它还有移动，边缘检测功能，因此继承gameobject

* Move控制球体移动，Velocity是矢量，会更新Position属性，如果到达了屏幕边缘，它的相应坐标（元素）会取反，如果球超出了底部边界，则不会改变方向（球超出了可视范围）
* Reset方便我们重置球

![1731488380271](/assets/img/blog/breakout/gif1.gif)

## window抽象

我想将窗口部分从program抽象出来，让program更简化一点

window窗口类，包括：

* 初始化glfw，窗口，glew，渲染配置
* while中逻辑
* 响应事件
* 销毁清理

## 碰撞

球体可以和屏幕边缘交互，但可以明显看到，当碰撞到砖块时没有任何交互效果

首先应该考虑如何判断和砖块是否碰撞？想到了ray_tracing中求交点的一种方式AABB

AABB(Axis-aligned Bounding Box)：轴对齐包围盒(边缘和坐标轴平行)（根据左上角点和右下角点（左上+size）定义）

由于场景中可能并非是方形物体（还有可能是三角形，圆形（玩家）等），因此用AABB包围盒（近似）可以简化求交点的过程

齐次要考虑和砖块碰撞会发生什么？包括砖块被销毁或反弹

//

定义一个新的Collision类

* CheckCollision检查两个game object（AABB-AABB）是否有重叠（第一个物体的最右侧是否大于第二个物体的最左侧并且第二个物体的最右侧是否大于第一个物体的最左侧；垂直的轴向与此相似），如果重叠返回真
* CheckCollision检查两个game object（AABB-圆形）是否有重叠

理解下面代码：核心思路就是找到AABB举例⚪的最近点，判断它到圆心的距离 < radius就相交
如何找到最近点closest呢？

首先获得⚪和AABB的center点，计算矢量差，通过clamp（2D的矢量表示将其x和y分量都限制在给定的范围内，如果分量在范围内不会更新，否则更新）即可找到最近点

```c++
GLboolean CheckCollision(BallObject &one, GameObject &two) // AABB - Circle collision
{
    // 获取圆的中心 
    glm::vec2 center(one.Position + one.Radius);
    // 计算AABB的信息（中心、半边长）
    glm::vec2 aabb_half_extents(two.Size.x / 2, two.Size.y / 2);
    glm::vec2 aabb_center(
        two.Position.x + aabb_half_extents.x, 
        two.Position.y + aabb_half_extents.y
    );
    // 获取两个中心的差矢量
    glm::vec2 difference = center - aabb_center;
    glm::vec2 clamped = glm::clamp(difference, -aabb_half_extents, aabb_half_extents);
    // AABB_center加上clamped这样就得到了碰撞箱上距离圆最近的点closest
    glm::vec2 closest = aabb_center + clamped;
    // 获得圆心center和最近点closest的矢量并判断是否 length <= radius
    difference = closest - center;
    return glm::length(difference) < one.Radius;
}   
```

在game中

* 新增DoCollisions函数（在update调用），会检查球体和每个砖块是否发生碰撞，如果发生碰撞，就将砖块的Destroyed = GL_TRUE，即不再绘制

![1731501601413](/assets/img/blog/breakout/Collision1.png)

## 碰撞处理

现在可以将彩色砖块销毁，还需要和砖块以及挡板交互，即碰撞反弹，我们要注意不能让物体重叠（需要移出AABB），且要考虑反弹速度方向

//

当球少量进入AABB时，应该将球移出AABB，确定需要将球从AABB中移出多少距离radius - abs(difference) == 侵入距离 == 需要移出距离

//

对于速度方向：逻辑很简单

* 如果球撞击AABB的右侧或左侧，它的水平速度（x）将会反转。
* 如果球撞击AABB的上侧或下侧，它的垂直速度（y）将会反转。

但是如何判断球体和AABB哪一侧碰撞呢？

点乘：我们通过clamp求得碰撞点的位置，创建北、南、西和东的四个矢量（正交化），分别与它们点乘获得夹角，并更新最大值（点乘结果越大，向量夹角越小），即可更新碰撞方向

//

Collision类：

* 添加新的VectorDirection函数，返回碰撞方向
* 会定义一个Direction 枚举（可读性和可维护性，安全性）
* 为了从Check***返回更多的碰撞信息，定义 std::tuple元表（是否发生碰撞，碰撞方向，侵入距离difference）

//

更新DoCollisions的逻辑

* 遍历每个砖块，如果砖块碰撞到球体，且如果砖块不是实心就销毁砖块
* 从碰撞信息获取碰撞方向

//

对于和挡板碰撞逻辑，我们不用检测碰撞方向（假设碰撞总是发生在挡板顶部），而是更新它的速度（撞击点距离挡板的中心点越远，则水平方向的速度就会越大，越趋向水平）

计算距离：球的中心与挡板中心的距离和挡板的半边长的百分比，反转它的y方向速度（往回弹）

新的速度：新的方向 * 速度
