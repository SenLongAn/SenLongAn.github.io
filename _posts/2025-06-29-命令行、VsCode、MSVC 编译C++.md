---
title: 命令行、VsCode、MSVC 编译C++
date: 2025-06-29 12:00:00 +0800
categories: [工具]
tags: []
math: true
---

# 编程环境相关名词

* GNU/Linux、GNU计划创建一套完全自由的操作系统，诞生GNU/Linux操作系统
* gcc、编译器,通常用于编译C
* g++，编译器,通常用于编译C++，
* 通常我们将两个编译器统称称为gcc编译器集合，它支持多种编程语言，包括C、C++、Objective-C、Fortran等

* Windows 操作系统
* MINGW，包括Windows特定头文件库文件和使用GNU工具集GCC编译器，以便为windows提供特定的GNU开发环境，而不需要第三方C运行时库MSVC，可编译C/C++语言
* MSVC、微软开发的第三方C运行时库/编译器，被Visual Studio IDE所集成，可编译C/C++语言

* LLVM构架编译器的框架系统,用于优化以任意程序语言编写的程序的构建、运行、空闲时间

* make构建生成器，当代码框架增大，需要大量源文件、各种库 时，每次编译重新编写编译命令，通过make的makefile配置文件指定项目构建过程，从而当发生代码更改，只需调用简单的命令即可
* cmake 构建生成器（为源码生成针对不同平台的配置文件），为了解决make的跨平台、不同编译器问题，和支持更多的功能，通过编写独立的CmakeList.txt文件，会选择系统环境和编译器来翻译CmakeList.txt，再进一步调用系统编译器构建文件

* IDE高度集成开发环境：集成了代码编辑器，编译器，调试器和图像化用户界面……Visual Studio

* 编辑器：可以理解为强大的记事本，Visual Studio Code，需要自己写配置文件，不易上手

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
  * 输入 hello运行程序
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
  * configuration.json：C/C++ 编译器配置

    ```c++
    {
        "configurations": [
            {
                "name": "Win32",
                "includePath": [
                    "${workspaceFolder}/**",
                    "${env:INCLUDE}"
                ],
                "defines": [
                    "_DEBUG",
                    "UNICODE",
                    "_UNICODE"
                ],
                "windowsSdkVersion": "10.0.22000.0",
                "compilerPath": "cl.exe",
                "cStandard": "c17",
                "cppStandard": "c++17",
                "intelliSenseMode": "windows-msvc-x64",
                "compilerPath": "D:\\……\\cl.exe"
            }
        ],
        "version": 4
    }
    ```

    * F! / ctrl+shift+p 打开命令调用面板，输入Edit Configurations(JSON)
  * task.json：定义构建任务

    ```c++
    {
        "version": "2.0.0",
        "tasks": [
            {
                "label": "build debug Win32",
                "type": "process",
                "command": "cl",
                "args": ["/Fe:.\\…….exe","/Fo:.\\","/MTd","/Zi", "/Od", "/EHsc",".\\.cpp"],//FE:可执行文件输出目录和文件名，FO构建阶段文件输出目录，MTD：运行时库链接方式，……，最后是要编译的文件
                "group": "build",
                "problemMatcher": [
                    "$msCompile"
                ]
            }
        ]
    }
    ```

    * F! / ctrl+shift+p 打开命令调用面板，输入Tasks:Configure Default Build Task
  * 现在可以编译了
    * Terminal->Run Task -> 选择刚才的Task

    ```c++
    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "(Windows) 启动",
                "type": "cppvsdbg",
                "request": "launch",
                "program": "…….exe",
                "args": [],
                "stopAtEntry": false,
                "cwd": "${workspaceFolder}",
                "environment": [],
                "externalConsole": false
            }
        ]
    }
    ```

  * launch.json：配置调试启动参数
    * 按Run->AddConfiguration  选择C++（Windows）
    * 出现没有用于调试JSON with comments的扩展​​，切换到cpp文件再按下F5
  * 现在可以运行了
    * 按下F5/Run->StartDebugging,即可在DEBUG CONSOLE看到输出
* 调试
  * 它自动帮助我们生成了pdb文件，因此可以调试
* settings.json：项目级设置
* 编译多个源文件
  * 新增源文件
  * 对tasks.json的要编译的文件新增文件
  * 编译
  * 运行
