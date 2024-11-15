---
title: VulkanTutorial（16`Depth buffering）
date: 2024-11-07 12:00:00 +0800
categories: [Vulkan]
tags: [Vulkan]     # TAG names should always be lowercase
math: true
---
# Depth buffering

之前顶点数据一直使用的2D坐标，这次我们使用3D数据，并且我们本次添加多个物体，然后进行深度测试

#### 3D数据

首先修改Vertex结构体，并修改attributeDescriptions的格式
接下来，更新顶点着色器以接受并转换3D坐标作为输入。别忘了以后重新编译它！
最后，更新vertexs顶点实际数据：

#### 多个物体

如果您现在运行应用程序，那么您应该看到与以前完全相同的结果，不过第一个目标已经完成，现在添加另一个四边形
让我们复制四边形顶点数据，然后将它们的z坐标变为-0.5（在下方），再修改indices数组

#### 深度测试

现在运行你的程序却发现，下面的方形绘制在上面的方形上方，因为我们后绘制下面的方形，有两种方法可以解决：

* 按深度从后到前对所有绘制调用进行排序（通常用于透明物体，因为如果按照正常深度测试绘制透明物体，深度较大并且后绘制的物体，会因为不会通过深度测试而丢弃，但正常来说，我们想要它显示到其他透明物体的后面）
* 使用深度缓冲区进行深度测试

深度缓冲区是一个额外的附件，它存储每个位置的深度，就像颜色附件存储每个位置的颜色一样

## buffer and attachment

对于buffer用于存储和在 CPU 和 GPU 之间**传输**（mapping）数据，而对于attachment负责**输出**和资源管理
shader会生成像素数据，并将这些数据写入到指定的attachment中，通过在vkRenderPassBeginInfo阶段绑定到swapchain的FrameBuffer（是一组Attachment（颜色/深度/模板）的集合），因此渲染结果实际上是被写入到了swapchain的image中，在渲染完成后，swapchain会负责将这些包含渲染结果的image按照特定的顺序和时机呈现到屏幕上

## GLM

GLM生成的透视投影矩阵将默认使用OpenGL深度范围-1.0到1.0。我们需要使用GLM_FORCE_DEPTH_ZERO_TO_ONE定义将其配置为使用Vulkan范围0.0到1.0。

## Depth image and view

#### Format

和颜色附件一样，深度附件也基于图像，但是我们要手动创建，
深度图像应与颜色附件相同的分辨率，但是要考虑如何指定它的格式，我们不需要从中读取数据，只需要有一个合理的精度

* VK_FORMAT_D32_SFLOAT：深度的32位浮点数
* VK_FORMAT_D32_SFLOAT_S8_UINT：深度和8位模板的32位有符号浮点数
* VK_FORMAT_D24_UNORM_S8_UINT：深度和8位模板的24位浮点数

模板组件用于模板测试，因此我们可以简单地使用VK_FORMAT_D32_SFLOAT格式，
添加一个新的函数findSupportedFormat（），它接受一个候选格式列表，对于每个格式，使用 vkGetPhysicalDeviceFormatProperties 函数获取该格式在物理设备上的特性
检查此特性是否满足特定的 tiling（图像平铺方式:VK_IMAGE_TILING_LINEAR（线性平铺）或 VK_IMAGE_TILING_OPTIMAL（最优平铺））和 features（格式特性标志）要求
从而选择到底使用那种深度图像格式是最好的

#### image and view

创建新的createDepthResources函数，首先调用findSupportedFormat（）找到format，调用已有的createImage创建深度图像，和调用createImageView创建图像视图
但是createImageView函数目前假设子资源总是VK_IMAGE_ASPECT_COLOR_BIT，所以我们需要转动那个字段转换成参数：

## renderpass，Framebuffer

我们不需要显式地将图像的布局转换为深度附件，因为我们将在renderpass中处理这个问题
我们现在要修改RenderPass以包含深度附件depthAttachment，还有subpass的reference
subpass只能使用一个depthAttachment，因此不用指定类似这样的colorAttachmentCount数量，
更新VkSubpassDependency结构体，引用附件
将depthImageView深度图像绑定到深度附件，还需要将调用移动到JumeFrameBuffers，以确保在实际创建深度图像视图之后调用它

## Clear values

我们有多个附件，因此要有对应的VkClearValue，转到recordCommandBuffer并创建一个VkClearValue结构体数组
VkClearValue表明每次渲染前，清除attachment为指定值，在Vulkan中，深度缓冲区中的深度范围为0.0到1.0，其中1.0位于远视图平面，0.0位于近视图平面

## Depth and stencil state

在pipeline中新增VkPipelineDepthStencilStateCreateInfo结构体，以便启用深度测试
depthTestEnable字段指定是否应将新片段的深度与深度缓冲区进行比较，以查看是否应丢弃它们
depthCompareOp字段指定为保留或丢弃片段而执行的比较
minDepthBounds和maxDepthBounds字段是用于可选的深度边界测试
并更新VkGraphicsPipelineCreateInfo结构体

## Handling window resize

当调整窗口大小时深度缓冲区的分辨率应更改
修改recreateSwapChain函数添加createDepthResources，
修改cleanupSwapChain函数添加vkDestroyImageView
![1730955678481](/assets/img/blog/vulkan/Depth.png)
