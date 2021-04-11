---
layout: post
author: "ooftf"
tags: Android
---

# UI优化
### 在系统框架下优化
    布局优化、使用代码创建、View缓存等。我们希望减少甚至省下渲染流水线里的某个耗时阶段
### 利用系统新特性
    使用硬件加速、RenderThread、RenderScript都是这个思路，通过系统一些新特性，最大限度压榨出性能
### 突破系统限制
    Facebook的Litho;Google的Flutter 
# 小知识点
    Flutter把Skia引擎直接集成到App中，就像一个游戏App一样           