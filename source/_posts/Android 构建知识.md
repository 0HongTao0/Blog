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

<!--more-->

##### Android 构建流程步骤

- 编译器（Compilers）将源代码（SourceCode）转换成 DEX 文件（Dalvik Executable，Dalvik 是运行在 Android 上面的虚拟机，.dex 文件包括运行在 Android 上的字节码），并且将其他内容转换成已编译资源。

- APK 打包器将 DEX 文件和已编译文件合并成单个 APK，不过要先进行对 APK 进行签名。

- APK 打包器使用调试（Debug）或发布（Release）密钥来对 APK 进行签名处理。（1）构建 Debug 版本，则 Android Studio 自动生成 Debug 的签名密钥。（2）构建 Release 版本则需要用户自己生成签名密钥。

- 在生成最终 APK 之前，打包器使用 zipalign 工具进行对 APK 优化。（减少运行内存占用，zipalign 是一个进行对未压缩数据对齐工具）

![Android 构建流程](https://github.com/0HongTao0/Blog/blob/master/pic/Android_%E6%9E%84%E5%BB%BA%E6%B5%81%E7%A8%8B.png?raw=true)

##### Android 构建的配置文件

首先我们新建的 Android 项目，通常 Android Studio 帮我们自动创建了几个 gradle 文件

![AndroidStudio 新建项目](https://github.com/0HongTao0/Blog/blob/master/pic/AndroidStudio%E6%96%B0%E5%BB%BA%E9%A1%B9%E7%9B%AE.jpg?raw=true)

- settings.gradle 文件（位于项目根目录），此 gradle 文件用于指示 Gradle 在构建应用时包含哪些模块（Module）在内。

  如下面代码包含 app 本身模块以及 alpha 模块

  ```groovy
  include ':app'
  include ':alpha'
  ```

- 顶级构建文件 build.gradle（位于项目根目录），此 gradle 文件适用于项目所有模块的构建配置

  ```groovy
  buildscript {
    // 下载远程依赖的仓库地址（远程仓库地址有JCenter, Maven Central, and Ivy，当然也可以用国内的某些镜像）
      repositories {
          google()
          jcenter()
      }
    // 指定需要用于构建项目的 Gradle 版本信息
      dependencies {
          classpath 'com.android.tools.build:gradle:3.1.1'
      }
  }
  // 配置项目中所有模块使用的远程仓库地址和依赖
  allprojects {
      repositories {
          google()
          jcenter()
      }
  }
  ```

- 模块级构建文件 build.gradle （位于模块的根目录下），此 gradle 文件用于配置所在模块的构建设置。

  ```groovy
  // 构建的插件                    
  apply plugin: 'com.android.application'
  
  //配置 android 应用程序的构建环境和使用环境
  android {
        // 指定使用哪个版本的 SDK 编译 app
      compileSdkVersion 27
      defaultConfig {
            //通常使用包名来定义 applicationId ，由main / AndroidManifest.xml文件中的package属性定义。
          applicationId "xyz.awqingnian.touchpull"
            // Android 设备要运行此 app 最小的 SDK 版本
          minSdkVersion 21
            // 指定用于测试此 app 的 SDK 版本
          targetSdkVersion 27
            // app 的版本号
          versionCode 1
            // app 的版本名字
          versionName "1.0"
          testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
      }
      buildTypes {
            //发布版本的配置信息
          release {
              minifyEnabled false //是否启用代码压缩（否）
              proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
          }
      }
  }
  // 模块中用到的第三方依赖
  dependencies {
      implementation fileTree(dir: 'libs', include: ['*.jar'])
      implementation 'com.android.support:appcompat-v7:27.0.2'
      implementation 'com.android.support.constraint:constraint-layout:1.0.2'
      testImplementation 'junit:junit:4.12'
      androidTestImplementation 'com.android.support.test:runner:1.0.1'
      androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
  }
  ```



| dependencies 配置 | 引用生命期                                                                                                                                 |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| implementation  | 依赖项在编译时对模块可用，并且仅在运行时对模块的消费者可用。 对于大型多项目构建，使用`implementation`而不是`api`/`compile`可以**显著缩短构建时间**，因为它可以减少构建系统需要重新编译的项目量。 大多数应用和测试模块都应使用此配置。 |
| api             | 依赖项在编译时对模块可用，并且在编译时和运行时还对模块的消费者可用。 此配置的行为类似于`compile`（现在已弃用），一般情况下，您应当仅在库模块中使用它。 应用模块应使用`implementation`，除非您想要将其 API 公开给单独的测试模块。      |
| compileOnly     | 依赖项仅在编译时对模块可用，并且在编译或运行时对其消费者不可用。 此配置的行为类似于`provided`（现在已弃用）。                                                                          |
| runtimeOnly     | 依赖项仅在运行时对模块及其消费者可用。 此配置的行为类似于`apk`（现在已弃用）。                                                                                            |



##### Gradle 属性文件

- gradle.properties ： 配置项目范围 Gradle 设置，例如 Gradle 后台进程的最大堆大小。

- local.properties ： 为构建系统配置本地环境属性，例如 SDK 安装路径。由于该文件的内容由 Android Studio 自动生成并且专用于本地开发者环境，因此您不应手动修改该文件，或将其纳入您的版本控制系统。
