---
title: VulkanTutorial（15`Texture Mapping）
date: 2024-11-06 12:00:00 +0800
categories: [Vulkan]
tags: [Vulkan]     # TAG names should always be lowercase
math: true
---
# Texture mapping

我们一直使用的用顶点数据着色（逐顶点），现在通过纹理映射去着色，基本步骤如下：

* ·创建由device内存支持的*图像对象*（image）
* ·使用图像文件中的像素*数据填充*（data->image）
* ·创建图像采样器*image sampler*(shader)
* ·添加*sampler descriptor*以从纹理中采样颜色（image->shader）

## Image layouts

这次自己创建一个图像对象VkImage，在这之前，首先创建暂存区并使用数据填充它，再复制到最终的图像对象，创建图像与创建缓冲区没有太大的不同，都是分配GPU内存和绑定内存
但是对于图像，我们必须考虑布局，这会影响像素在内存中的组织方式：

* VK_IMAGE_LAYOUT_UNDEFINED：不关心布局undefined
* VK_IMAGE_LAYOUT_PRESENT_SRC_KHR：演示
* •VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL：颜色附件
* •VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL：作为源传输操作
* •VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL：作为目的地传输操作
* •VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL：着色器只读布局

## Image library

有许多库可用于加载图像，您甚至可以编写自己的代码来加载BMP和PPM（比如ray tracing教程中）等简单格式。
本次使用stb_image库，它只有一个简单的h和cpp文件，将它包含到你的项目
这个宏包含函数体

```
#define STB_IMAGE_IMPLEMENTATION
#include <stb_image.h>
```

❌vs“找不到 **.dll，无法执行代码
最方便的方式，将dll与exe文件放在同一目录

## Loading image data

创建新函数createTextureImage（）
将任意一个大小为512 x 512像素的图像（stb_image库支持JPEG，PNG，BMP和GIF格式），放置在项目中
stbi_load（）加载图像，STBI_rgb_alpha值强制使用alpha通道加载图像，返回的指针是**像素数组**中的第一个元素，像素以每像素4字节的方式逐行布局，总共为texWidth * texHeight * 4个值

## Staging buffer

和其他缓冲区一样，在GPU创建stagingBuffer分级缓冲区并分配内存，通过vkMapMemory将pixels数据填充到缓冲区内存中，不要忘记通过stbi_image_free清理原始像素
![1730875919355](/assets/img/blog/vulkan/Buffering%20relationship.png)

## VkImage

虽然可以通过description的方式，在着色器访问缓冲区中的像素值，但最好使用VkImage来实现此目的。因为图像对象将使它更容易和更快地检索颜色，允许我们使用2D坐标
VkImageCreateInfo，vkCreateImage

* imageType字段中指定的图像类型告诉Vulkan图像中的纹素将使用哪种坐标系进行寻址，可以创建1D、2D和3D图像
* extent字段指定图像的尺寸，基本上就是每个轴上有多少个纹理元素
* format 应该为纹理元素使用与缓冲区中像素相同的格式，否则复制操作将失败
* tiling 具有以下两个值之一
  * VK_IMAGE_TILING_LINEAR：纹理元素以行为主的顺序布局，如我们的像素阵列
  * VK_IMAGE_TILING_OPTIMAL：纹理元素以实现定义的顺序进行布局，以实现最佳访问
* initialLayout只有两个可能的值：
  * VK_IMAGE_LAYOUT_UNDEFINED：不可用于GPU，并且第一次转换将丢弃纹素。
  * VK_IMAGE_LAYOUT_PREINITIALIZED：GPU无法使用，但第一次转换将保留纹理元素。
* usage字段的语义与缓冲区创建期间的语义相同，该图像将用作缓冲区复制的目的地
* samples标志与multisystem多重采样相关，先使用1个采样点
  然后就像普通vkbuffer一样，为VkImage分配内存，与为缓冲区分配内存的工作方式完全相同
  使用vkGetImageMemoryRequirements而不是vkGetBufferMemoryRequirements，并使用vkBindImageMemory代替vkBindBufferMemory。

## Layout transitions

首先抽象出两个包含cmd命令的函数，因为要多次使用，并将原来的copyBuffer函数简化
beginSingleTimeCommands包括vkBeginCommandBuffer
endSingleTimeCommands包括vkEndCommandBuffer，vkQueueSubmit
//
我们在buffer->image之前首先应该处理布局过渡，
最常见方法之一是**VkImageMemoryBarrier屏障**转换图像布局，它即能资源状态转换，又能保证同步执行（任务不能重叠），防止数据竞争等问题

* oldLayout和newLayout两个字段指定布局
* 如果您使用屏障来转移队列家族所有权，那么这两个字段应该是队列家族的索引。如果你不想这么做，它们必须设置为VK_QUEUE_FAMILY_IGNORED
* image和subresourceRange指定受影响的图像和图像的特定部分
  // barrier masks遮罩
* srcAccessMask
  * = 0;传输写入不需要等待任何东西
  * = VK_ACCESS_TRANSFER_WRITE_BIT。这表示在源阶段应该等待传输写入
* dstAccessMask = VK_ACCESS_SHADER_READ_BIT。这表明在目的阶段图像将被着色器读取
* sourceStage  VK_PIPELINE_STAGE_TRANSFER_BIT，这表明在传输阶段对图像执行操作
* destinationStage = VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT在片段着色器阶段图像将被使用

vkCmdPipelineBarrier用于在命令缓冲区中插入一个管线屏障
第二个参数指定了应该在屏障之前发生的操作在哪个流水线阶段发生。第三个参数指定操作将在屏障上等待的流水线阶段

## CopyBufferToImage

图像布局转换后就可以调用vkCmdCopyBufferToImage

* bufferOffset指定缓冲区中像素值开始的字节偏移量。
* bufferRowLength和bufferImageHeight字段指定像素在内存中的布局方式
* 第四个参数指示图像当前使用的布局

## Calling function

现在我们已经拥有了完成纹理图像设置所需的所有工具，首先应将图像布局转换为VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL，然后CopyBufferToImage，最后转换为VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL以便能从着色器中的纹理图像开始采样

## Texture image view

我们知道图像是通过image view而不是直接访问的，
首先抽象出createImageView函数，修改原有的createImageViews函数简化
不要忘记vkDestroyImageView

## sampler

着色器如何从图像中读取纹理元素?
着色器可以直接从图像中读取纹素，但当它们用作纹理时，通常通过sampler访问
sampler将应用**过滤**（如何从图像采样像素颜色值）**转换**（决定了当你试图通过它的寻址模式读取图像外部的纹理元素时会发生什么）来计算检索到的最终颜色
首先createTextureSampler，VkSamplerInfo

* magFilter和minFilter字段指定如何插入放大或缩小的纹素
* addressMode字段指定每个轴的寻址模式，这些轴被称为U，V和W，而不是X，Y和Z
  * VK_SAMPLER_ADDRESS_MODE_REPEAT：在超出图像尺寸时重复纹理。
  * •vk_sampler_address_mode_mirrorred_repeat：类似于repeat，但在超出维度时颠倒坐标以镜像图像。
  * •VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_EDGE：取图像尺寸之外最接近坐标的边缘的颜色。
  * •VK_SAMPLER_ADDRESS_MODE_MIRROR_CLAMP_TO_EDGE:像clamp到边一样，但使用与最近边相反的边。
  * •VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_BORDER：返回纯色当采样超出图像的尺寸时。
* **anisotropy**Enable 指定是否应使用**各向异性过滤**，没有理由不使用它
* maxAnisotropy 限制可用于计算最终颜色的纹素样本数量。值越小，性能越好，但质量越低，为了弄清楚我们可以使用哪个值，通过vkGetPhysicalDeviceProperties函数，返回VkPhysicalDeviceProperties
* borderColor字段指定当使用箝位到边界寻址模式在图像之外采样时返回的颜色。可以以float或int格式返回黑色、白色或透明
* unnormalizedCoordinates字段指定要使用哪个坐标系来处理图像中的纹素
* compareEnable 如果启用了比较功能，则纹理元素将首先与某个值进行比较
* mipmapping过滤器
  不要忘记vkDestroySampler
  //
  如果现在运行程序，validating layout将告诉你vkCreateSampler()：VkSamplerCreateInfo-anisotropyEnable is VK_TRUE but the samplerAnisotropy feature was not enabled.是VK_TRUE，但samplerAnisotropy特征未启用。
  这是因为各向异性过滤实际上是一个可选的设备功能，我们需要更新isDeviceSuitable函数以检查它是否可用
  vkGetPhysicalDeviceFeatures

## Updating the descriptors

添加新的描述符，使着色器可以通过一个采样器对象访问图像资源
我们将首先修改描述符布局、描述符池和描述符集，以包含新的uniform资源
之后，我们将添加纹理坐标到顶点数据，并修改片段着色器从纹理读取颜色，而不仅仅是插值计算顶点颜色
//
首先修改VkDescriptorSetLayout函数，确保设置stageFlags
修改createDescriptorPool，变为poolSizes数组
createDescriptorSets的descriptorWrites也一样，修改为数组

## Texture coordinates，Shaders

修改顶点数据结构体Vertex ，添加texCoord，和getAttributeDescriptions顶点描述
还有我们的实际数据vertices
最后一步是修改着色器以从纹理中采样颜色，
在顶点着色器，我们添加新的layout(location = 2)以便从顶点数据传入到顶点着色器，通过out输出到片段着色器
片段着色器通过in 接收纹理坐标，添加layout(binding = 1) uniform sampler2D
![1730894319500](/assets/img/blog/vulkan/Texture.png)

# vs快捷键

发现一个超好用的功能，按住ctrl放在词上，当变蓝后点击，会转到定义/链接出
