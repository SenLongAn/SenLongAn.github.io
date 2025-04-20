---
title: 个人博客网站搭建
date: 2024-11-05 12:00:00 +0800
categories: [工具]
tags: []     # TAG names should always be lowercase
math: true
---
# 简述

传统的企业网站搭建方式通过云服务器（阿里云/腾讯云/华为云购买） + 域名 + 框架

云服务器（Elastic Compute Service弹性计算服务，ECS）是一种虚拟化服务器，利用虚拟化技术，将物理服务器的计算资源（如CPU、内存、存储等）封装成一个或多个独立的虚拟环境，用户可以通过互联网远程访问和使用这些资源

域名（Domain Name）是由一串用点分隔的名字组成的，Internet上某一台计算机或计算机组的名称，相当于标识符

但是对于轻量级的个人网站我们使用一种简单免费的方式：Github.io + Jekyll框架

GitHub Pages站点，是GitHub提供的一个***静态网站托管服务***,它允许用户将项目的静态文件（如HTML、CSS和JavaScript等）部署到一个个人网站上

这种方式不需要云服务器，而是利用Pages托管生成静态网站，它提供了Github.io的二级域名，因此也不需要购买域名

Jekyll是一种适用于博客的、静态网站生成引擎，它使用一个***模板***目录作为网站布局的基础框架，最终生成一个完整的静态Web网页站点（不用写html）

这种方式满足我们博客、个人主页、在线简历、项目演示等这些简单需求

# Jekyll配置

[chirpy模板官方文档](https://chirpy.cotes.page/)

[jekyll安装官方文档](https://jekyllrb.com/docs/)

查看所需环境配置，若没有则安装：

* √Ruby (ruby -v)（服务器端脚本语言）

  * √到官网下载version >=3.3.22版本
  * [Downloads地址（选择Ruby + Devkit 的)](https://rubyinstaller.org/downloads/)
  * √设置环境变量，path中配置\bin目录
  * √RubyGems (gem -v)（Ruby的一个包管理器）
    * 升级gem：gem update --system
  * √(DevKit（jekyll --v）（bundler -v）（windows平台下编译和使用本地C/C++扩展包的工具)
    * gem install jekyll bundler安装 jekyll 和 bundler
  * √Rdiscount  -> gem install rdiscount   (用来解析Markdown标记的包)
* √Gcc & make (gcc -v | g++ -v | make -v )（Gcc编译器（C语言编译器），G++编译器（C++编译器），Make构建工具的版本信息（需要将mingw32-make.exe拷贝一份命名为make.exe））
* √Git (git --v)分布式版本控制系统，Github提供了Git代码仓库托管（仅支持git，相当于云端存储功能）及基本的Web管理界面，因此得名GitHub

# Github Pages个人站点

[使用这个模板](https://chirpy.cotes.page/posts/getting-started/)

√单击 Use this template 按钮，然后选择 Create a new repository

√换模板：我更新了模板使用fork

个人站点：使用自己的用户名，每个用户名下面只能建立一个，必须符合这样的规则username/username.github.io

√clone项目

* √修改Gemfile文件source **"https://mirrors.tuna.tsinghua.edu.cn/rubygems"**替换为清华源（有时反而不好使，换回去）
* √打开_config.yml文件，填写url为仓库名

√repository->setting->Pages->Build and deployment中选择Github Actions（等待成功部署才会更新）

√git commit提交本地修改

√打开你的网站username/username.github.io

### 本地运行

√Gemfile文件包含一系列gem命令，用于安装相关库，打开命令行执行bundle install即完成安装

* ❌An error occurred while installing wdm (0.1.1), and Bundler cannot continue.
  * 解决方式：在gemfile注释掉wdm一行

√

```c++
bundle exec jekyll s
```

启动Jekyll服务器

在本地执行，几秒钟后，本地服务器将在 [http://127.0.0.1:4000](http://127.0.0.1:4000/) 可用bundle exec jekyll s

这样就能**实时**查看了，而不用每次都push，全部修改好后再push

# 配置个人主页

##### 基本配置都在_config.yml修改

* url: 网址 [http://USERNAME.github.io](https://link.zhihu.com/?target=http%3A//username.github.io/)
* lang: [语言(默认en英文)]
* title: 侧边栏名称
* timezone:[时区(中国只能选上海)](https://zones.arilyn.cc/)
* tagline：标语
* github: 左下角图标地址
* avatar: 头像
  * 在本地Pages的/assets/img/avtar/此路径下，传入图片
  * 修改config.yml：avatar:/assets/img/avtar/…….png

##### 更改网站的图标

[在线生成icon网站](https://realfavicongenerator.net/)

替换原有目录的网站图标assets/img/favicons/

这里有可能替换失败，查找原有图标所有相关目录，替换为新的

##### 写博客

* 文件名必须为YYYY-MM-DD-TITLE.EXTENSION的格式，并将其放在根目录的 `_posts`中，其中 `EXTENSION`必须是 `md`和 `markdown`之一

```
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]     # TAG names should always be lowercase
---
```

* 在文章的顶部填写以下的前提（可选），比如这样示例，然后就可以用markdown写博客了

```
---
title: 我的第一篇博客文章 
date: 2024-11-05 12:00:00 +0800
categories: [随笔博客, 简述]
tags: [随笔]     # TAG names should always be lowercase
math: true
---
```

* title是文章的名称
* categories是标签分类
* tags是文章的标签
* math可以使用数学公式

##### 图片

如下形式
`![](相对路径)`

##### 代码段

```python
import gpt
def HelloWorld():
    print("hello world")
```

##### 提示信息

> An example showing the `tip` type prompt.
> {: .prompt-tip }

> An example showing the `info` type prompt.
> {: .prompt-info }

> An example showing the `warning` type prompt.
> {: .prompt-warning }

> An example showing the `danger` type prompt.
> {: .prompt-danger }

##### 评论区

Utterances:

* [这里选择repository为它安装](https://github.com/apps/utterances)
* [打开这里方便配置评论区](https://utteranc.es/)
* 复制生成的代码区，放在你的博客中（你想要添加评论的位置）
* 修改_config.yml的comments部分（你也可以选择其他的方式）

或者使用giscus和Utterances几乎没有差异，我使用giscus

##### 更新repository

我们可能本地没有错误，但是对于网站可能有错误，
在push后，打开GitHub的actions，点进去当前的更新，里面的内容会告诉你为什么会失败（如果失败博客不会成功部署，即不会更新）

##### 访问量

不蒜子
