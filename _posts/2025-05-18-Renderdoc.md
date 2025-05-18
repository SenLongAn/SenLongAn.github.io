---
title: (1`Renderdoc)
date: 2025-05-18 12:00:00 +0800
categories: [工具]
tags: []
math: true
---
# Renderdoc

图形调试器，支持多个操作系统上的多个图形API

### 链接和调试

* UE_Editor
  * plugin:
    * 启动engine editor
    * 勾选Enabled RenderDoc Plugin 并 restart
    * 点击右上角renderdoc按钮
    * 点击capture frame 捕获帧
  * Attach：
    * 启动engine editor
    * renderdoc -> Attach to Running Instance -> engine editor -> connect to app
    * 点击capture frame 捕获帧
* UE_Game:
  * exe
    * 打包exe
    * Launch：
      * 运行exe
      * alt + f12 捕获帧（自动打开renderdoc）
    * Attach：
      * 运行exe
      * renderdoc -> Attach to Running Instance -> exe -> connect to app
      * 点击capture frame 捕获帧
  * apk
    * 数据线链接
    * apk install 到设备（手机/头显）上
    * renderdoc左下链接设备，状态变为 蓝色链接图标 + replay context: device name 即链接成功
    * renderdoc -> Launch Application -> executable path -> ……activity
    * 点击capture frame 捕获帧

捕获后双击下方的captures collected的某一帧可以调试，如果capture可以捕获多个帧，右键它们可以delete/框选后按下delete

注：在capture frame按钮亮起却无法捕获的情况，可以点击cycle active window 切换活动窗口

### renderdoc窗口

* pipeline state管线状态
  * 展示渲染流水线，其中未使用的阶段用灰色表示，选中的阶段被红色框包裹
  * VTX顶点数据  VS顶点着色器 TCS曲面细分着色器 TES曲面细分评估着色器 GS几何着色器 RS光栅化 FS片段着色器 FB帧缓冲 CS计算着色器
  * 对于每个阶段，点击go/view箭头图标，跳转到其他窗口，比如纹理，shader，buffer……
* event browser事件浏览器
  * 按照时间顺序（EID）列举 渲染捕获帧时的 事件（操作）或API
  * 选择的事件/api 将高亮显示并在左侧有绿色旗帜
  * 上方可以filter过滤搜索
  * ctrl + f -> 通过名字/EID find event
  * 点击黄色图标添加标签，这样就可以快速跳转到标记位置
* timeline时间轴
  * 是event browser水平方式展示
  * 每个事件/API有相同的时间，选择某个时间将在上方有绿色旗帜
  * 点击每个层展开/折叠，每个事件作为层的叶节点，蓝色表示，选择的变为绿色
  * 在下方有三角形标记，不同颜色代表当前纹理 只读/只写，可读可写
* api inspector API检查器
  * 在选择某个api时被更新，显示了API调用时的形参和实参
* texture viewer纹理查看器
  * 显示当前绑定的渲染目标，
  * 右侧的thumbnails缩略图，当前绑定的输入/输出（颜色/深度）目标
  * 上方toolbar可以按照通道，范围显示
  * 中间main texture display，可以滚轮缩放视图，它显示thumbnails中选择的目标
  * 右键纹理可以拾取点击的像素值,在右下的pixel context展示
  * 下方state bar状态栏，包含当前选择的纹理信息（纹理尺寸，深度，MSAA 样本数量和质量，格式……）
* Mesh Viewer网格视图
  * 在此窗口选择不同的阶段展示mesh输入输出
  * 可以查看顶点，索引，位置，
  * 预览mesh，点击某个顶点数据，在预览窗口以红色高亮渲染（highlight vertices要启用）
  * 可以选择visualisation选择mesh渲染的方式，默认是线框渲染
* Resource Inspector资源检查器
  * 捕获帧的所有资源，和资源的信息，对于粗体并有灰色链接图标🔗为资源，可以点击打开
  * resource list展示所有资源，双击选中
  * related resources相关资源，对于所选的资源，展示相关的父资源子资源
  * usage in frame 显示此资源被哪些EID使用，和usage使用方式
  * Resource Initialisation Parameters列出初始化资源的api
* Shader Viewer着色器视图
  * 用于显示、编辑和调试着色器，可以从pipeline打开
  * 此处显示shader源码
  * input/output signature 显示shader输入输出，包含名字，类型，mask……
* Buffer Viewer缓冲区查看器
  * 可以通过pipeline的go打开
