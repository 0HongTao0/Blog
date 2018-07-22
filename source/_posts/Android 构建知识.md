---
title: Android 构建知识

date: 2018-07-21 09:16:55

tags:



---

##### 简单介绍一下 [Gradle Build Tool](https://gradle.org/)

- 可以帮助团队更快地构建，自动化和交付更好的软件。

- 使用一种基于 Groovy 语言来**声明项目设置**。

- 支持多语言构建：Java,  C++, Python, Android , IOS 等等。

- Gradle 的优势：自动处理 dependencies 关系（Maven 概念），自动处理部署问题（Ant 概念），可以使用条件判断写法（Groovy语言）

##### Android 构建流程步骤

- 编译器（Compilers）将源代码（SourceCode）转换成 DEX 文件（Dalvik Executable，Dalvik 是运行在 Android 上面的虚拟机，.dex 文件包括运行在 Android 上的字节码），并且将其他内容转换成已编译资源。

- APK 打包器将 DEX 文件和已编译文件合并成单个 APK，不过要先进行对 APK 进行签名。

- APK 打包器使用调试（Debug）或发布（Release）密钥来对 APK 进行签名处理。（1）构建 Debug 版本，则 Android Studio 自动生成 Debug 的签名密钥。（2）构建 Release 版本则需要用户自己生成签名密钥。

- 在生成最终 APK 之前，打包器使用 zipalign 工具进行对 APK 优化。（减少运行内存占用，zipalign 是一个进行对未压缩数据对齐工具）

![Android 构建流程](https://github.com/0HongTao0/Blog/blob/master/pic/Android_%E6%9E%84%E5%BB%BA%E6%B5%81%E7%A8%8B.png?raw=true)



##### Android 构建的配置文件
