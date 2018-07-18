---
title: Android 屏幕适配
date: 2018-07-06 12:14:52
tags:
---

- 屏幕适配

  1. 屏幕尺寸（inch）：屏幕的对角线（5寸，6寸）

  2. 分辨率（px）：1920*1080 （像素点）

  3. 屏幕密度（dpi）：一英寸长度中可显示输出的像素个数

  4. 三者关系 
  ![三者关系](https://mmbiz.qpic.cn/mmbiz_png/5EcwYhllQOgM19n6iawpWQRCfcibxicoBYG51prmqwNCLAVALyK5Rhv4uSbrU5FQKQL6bZI3iaibTJaz3NMpEQ8zWAA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)
<!--more-->
- 单位之间的转换

  1. px = density * dp

  2. density = dpi / 160

  3. px = dp * (dpi / 160)

- [Android 官方给出的屏幕适配](https://developer.android.com/guide/practices/screens_support)

  1. wrap_content 和 match_parent

  2. 按照屏幕尺寸加载不同的资源文件

     _ldpi_（低）~120dpi

     mdpi_（中）~160dpi

     _hdpi_（高）~240dpi

     _xhdpi_（超高）~320dpi

     _xxhdpi_（超超高）~480dpi

     _xxxhdpi_（超超超高）~640dpi

- [其他方案]([https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA](https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA)

  1. 修改源码 density
