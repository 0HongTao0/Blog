title: Android 基础
date: 2018-11-30 14.00
tags:

------

### Content Provider

- [Content Provider 的一些理解](https://www.jianshu.com/p/c70ae80cf64d)
  1. 主要用于不同程序之间实现数据共享的功能，还可以通过 Content Provider 来实现跨进程之间的通信。在 Android 系统自带有很多 Content Provider（通信录，短信等）。
  2. 在 Content Provider 的 回调方法中， onCreate() 方法是运行在主线程（UI线程）中，在 AMS 通过 ActivityThread 创建并回调 onCreate() 方法，不能再 onCreat() 方法中耗时操作。其他方法 query(), insert(), update(), delete() 等方法都是运行在应用 程序的 Binder 维护的线程池中，所以在其他方法耗时操作不会阻塞主线程。
