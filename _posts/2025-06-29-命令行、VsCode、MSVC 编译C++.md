---
title: 命令行、VsCode、MSVC 编译C++
date: 2025-06-29 12:00:00 +0800
categories: [工具]
tags: []
math: true
---

# 名词

* GNU：GNU计划创建一套完全自由的操作系统，他们几乎完成了所有必要的用户空间工具，但缺少内核
* Linux：内核出现，和GNU组合在一起，形成完整可用的操作系统，称为Linux
* GNU 编译器集合
  * gcc、编译器,通常用于编译C
  * g++，编译器,通常用于编译C++
* Windows 操作系统
* MinGW：通过Windows特定头文件库文件和GNU编译器集合 形成的开发环境，也就是GNU 编译器集合 + Window内核，可以理解为Window的移植版本
* MSVC：微软开发的 C/C++ 编译器工具链，包括编译器和 C/C++ 运行时库, 被VS集成

* make构建生成器，当代码框架增大，需要大量源文件、各种库 时，每次编译重新编写编译命令，通过make的makefile配置文件指定项目构建过程，从而当发生代码更改，只需调用简单的命令即可
* cmake 构建生成器（为源码生成针对不同平台的配置文件），为了解决make的跨平台、不同编译器问题，和支持更多的功能，通过编写独立的CmakeList.txt文件，会选择系统环境和编译器来翻译CmakeList.txt，再进一步调用系统编译器构建文件

* IDE高度集成开发环境：集成了代码编辑器，编译器，调试器和图像化用户界面……Visual Studio

* 编辑器：可以理解为强大的记事本，如Visual Studio Code

* 运行时库：支持程序运行的基本函数的集合

# 命令行_MSVC_C++

* MSVC
  * 方案一：下载安装Visual Studio 和安装 “使用 C++ 进行桌面开发”工作负载
  * 方案二：安装Visual Studio 命令行生成工具：VS官网 -> 所有下载 -> 用于Visual Studio的工具 -> Visual Studio …… 生成工具
* MSVC环境
  * MSVC有极为复杂的环境，默认情况在命令提示窗口中不能使用MSVC，VS准备了快捷方式：开发人员命令提示Developer Command Prompt ，它为命令行生成了环境
  * 在启动的命令行输入cl，验证是否设置正确
* 编译一个源文件
  * cd到源文件目录
  * notepad hello.cpp创建一个hello.cpp文件
  * 打开文件输入简单的测试代码，保存
  * cl /EHsc hello.cpp 来编译程序（/EHsc选项用于启用标准 C++ 异常处理行为）
  * hello.exe 运行程序
* 编译多个源文件
  * cl /EHsc file1.cpp file2.cpp file3.cpp……
* 更改可执行文件名
  * cl /EHsc file1.cpp file2.cpp file3.cpp /link /out:program1.exe
* 若要自动捕获更多编程错误（建议使用 /W3 或 /W4 警告级别选项进行编译）
  * cl /W4 /EHsc file1.cpp file2.cpp file3.cpp /link /out:program1.exe

# VsCode_MSVC_C++

* 完成上面MSCV的配置
* 安装Visual Studio Code的插件：C/C++
* 从Developer Command Prompt启动带有MSVC环境的vscode
* 新建项目文件夹（英文）用vscode打开
* 新建项目文件…….cpp，输入简单的测试代码，保存
* 配置项目
  * c_cpp_properties.json：

    ```c++
    {
        "configurations": [
            {
                "name": "VulkanTutorial",
                "compilerPath": "……cl.exe",
                "cppStandard": "c++17",
                "includePath": [
                    "${workspaceFolder}/**",
                    "D:\\Vulkan\\VulkanSDK\\1.3.296.0\\Include",
                    "${workspaceFolder}/vendor/GLFW/include/",
                    "${workspaceFolder}/vendor/GLM/"
                ]
            }
        ],
        "version": 4
    }
    ```

    * 配置C/C++扩展
      * version版本固定为4
      * name项目名，任意字符串
      * compilerPath编译器路径
      * cStandard/cppStandard C/C++标准
        * "cppStandard": "c++11"
        * "cppStandard": "c++14" 
        * "cppStandard": "c++17"
        * "cppStandard": "c++20"
        * "cppStandard": "c++23"
      * includePath头文件搜索路径，${workspaceFolder}/**表示当前工作区下的所有子目录（可选）
      * intelliSenseMode智能代码补全辅助工具的模式（可选）
        * "intelliSenseMode": "windows-gcc-x64"    // GCC on Windows
        * "intelliSenseMode": "windows-msvc-x64"   // MSVC on Windows  
        * "intelliSenseMode": "linux-gcc-x64"      // GCC on Linux
        * "intelliSenseMode": "macos-clang-x64"    // Clang on macOS
      * defines预处理器宏定义（可选）
      * windowsSdkVersion版本（可选）
    * 创建方式：F! / ctrl+shift+p 打开命令调用面板，输入Edit Configurations(JSON)
  * task.json：

    ```c++
    {
      "version": "2.0.0",
      "tasks": [
        {
          "label": "MyTask",
          "type": "process",
          "command": "cl.exe",
          "args": [
            "/EHsc",
            "/Fe:bin/main.exe",
            "/Fo:bin/",
            "/Fd:bin/vc140.pdb",
            "/Zi",
            "/Od",
            "/MDd",
            "/I", 
            "D:\\Vulkan\\VulkanSDK\\1.3.296.0\\Include",
            "/I",
            "${workspaceFolder}/vendor/GLFW/include/",
            "/I",
            "${workspaceFolder}/vendor/GLM", 
            "*.cpp",
            "/link",
            "/LIBPATH:D:\\Vulkan\\VulkanSDK\\1.3.296.0\\Lib",
            "/LIBPATH:${workspaceFolder}/vendor/GLFW/lib-vc2022",
            "vulkan-1.lib",
            "glfw3.lib",
            "user32.lib",
            "gdi32.lib",
            "shell32.lib",
            "/NODEFAULTLIB:MSVCRT"
          ],
          "group": "build"
        }
      ]
    }
    ```

    * 配置构建器
      * version版本号，只有2.0.0
      * label任务名称，任意字符串
      * type任务类型
        * cppbuild:C/C++专用构建任务，自动配置调试信息
        * shell:执行shell命令.命令行任务
        * process:执行外部进程,直接运行可执行文件
      * command编译命令:
        * "command": "cl.exe"                    // MSVC编译器
        * "command": "g++"                       // GCC编译器  
        * "command": "clang++"                   // Clang编译器
        * "command": "cmake"                     // CMake
        * "command": "C:/MinGW/bin/gcc.exe"      // 完整路径
      * args命令行参数：MSVC：
        * /EHsc // 异常处理
        * /Fe:  //输出目录和可执行程序名
        * /Fo:  //中间文件目录
        * 链接方式
          * MT：多线程静态链接(静态库)
          * MTd：多线程调试静态链接，启用调试功能
          * MD：多线程动态链接（动态库）
          * MDd：多线程调试动态链接，启用调试功能
        * 源文件：
          * 列出每个文件的目录和文件名
          * 使用 目录/*.cpp，可以匹配此目录下所有文件
      * group任务分组，决定快捷键行为
        * build，有专用快捷键，用于Ctrl+Shift+B 构建
        * test测试任务组
        * 无分组，只能手动运行
      * problemMatcher解析编译器输出（可选）
      * options工作目录（可选）
    * VSCode配置文件的变量索引
      * ${workspaceFolder} - 工作区根目录 VS Code 中打开的文件夹目录 （通常是项目的位置）
      * ${workspaceFolderBasename} - 没有任何斜杠 (/)的 VS Code 中打开的文件夹目录
      * ${file} - 目前打开文件的绝对位置
      * ${relativeFile} - 目前打开文件相对于 workspaceFolder 的相对位置
      * ${fileBasename} -  目前打开文件的文件名（有拓展名，如： main.cpp）
      * ${fileBasenameNoExtension} - 目前打开文件的出去拓展名的文件名（无拓展名， 如： main.cpp）
      * ${cwd} - task runner的工作目录
      * ${fileDirname} - 目前打开文件的目录位置
      * ${fileExtname} - 目前打开文件的拓展名
      * ${lineNumber} - 文件中目前被选择的行数
      * ${selectedText} - 文件中目前被选择的内容
    * 创建方式：F! / ctrl+shift+p 打开命令调用面板，输入Tasks:Configure Default Build Task
    * 编译：Terminal->Run Task -> 选择刚才的Task
  * launch.json：

      ```c++
      {
          "version": "0.2.0",
          "configurations": [
              {
                  "name": "MyProject",
                  "type": "cppvsdbg",
                  "request": "launch",
                  "program": "${workspaceFolder}/bin/main.exe"
              }
          ]
      }
      ```

    * 配置运行器
      * name 任意字符串
      * type 指定调试器类型
        * "cppvsdbg" - Windows 上使用 Visual Studio 调试器
        * "cppdbg" - 使用 GDB/LLDB (MinGW, Linux, macOS)
        * "node" - Node.js 调试
      * request 调试请求类型
        * "launch" - 启动新程序进行调试
        * "attach" - 附加到已运行进程
      * program要调试的可执行文件路径
      * args传递给程序的命令行参数（可选）
    * 创建方式：
      * 按Run->AddConfiguration  选择C++（Windows）
      * 出现没有用于调试JSON with comments的扩展​​，切换到cpp文件再按下F5
    * 运行：按下F5/Run->StartDebugging,即可在DEBUG CONSOLE看到输出
* 调试
  * 生成pdb文件：在tasks.json中，/Zi生成调试信息文件，/Od禁用优化便更好地进行调试
* 编译多个源文件:新增源文件,在tasks.json中新增文件
* 使用库
  * c_cpp_properties.json中的includePath添加库的头文件目录
  * tasks.json中args中/I，添加库的头文件目录，/link，/LIBPATH:，添加库文件目录，还需要使用lib文件的名称
