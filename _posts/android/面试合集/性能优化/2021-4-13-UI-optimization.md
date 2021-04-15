---
layout: post
author: "ooftf"
tags: Android
---

## UI优化
[UI 优化](https://blog.csdn.net/freekiteyu/article/details/77862670)
### 在系统框架下优化
  布局优化、使用代码创建、View缓存等。我们希望减少甚至省下渲染流水线里的某个耗时阶段
#### 布局优化
* 避免过度绘制
  检测过度绘制：开发者选项 -> 调试GPU过度绘制，来打开该功能，来查看是否出现了过度绘制。
  * 原色：没有过度绘制
  * 蓝色：过度绘制1次
  * 绿色：过度绘制2次
  * 粉色：过度绘制3次
  * 红色：过度绘制4次或更多

  布局视图树扁平化
  1. 移除嵌套布局
  2. 使用merge、include标签
  3. 使用性能消耗更小布局（ConstraintLayout）
* 去除不必要的背景色
  1. 设置窗口背景色为通用背景色，去除根布局背景色。
  2. 若页面背景色与通用背景色不一致，在页面渲染完成后移除窗口背景色
  3. 去除和列表背景色相同的Item背景色
* 减少透明色，即alpha属性的使用
  1. 通过使用半透明颜色值(#77000000)代替
* 其他
  1. 使用ViewStub标签，延迟加载不必要的视图
  2. 使用AsyncLayoutInflater异步解析视图

常见的检测工具有：
1. Hierarchy Viewer（分析Ui性能）
2. 手机开发者自带GPU呈现模式（考核Ui性能）
3. TraceView （代码层面分析性能问题）
### 利用系统新特性
  使用硬件加速、RenderThread、RenderScript都是这个思路，通过系统一些新特性，最大限度压榨出性能
### 突破系统限制不再使用原生View
  Compose;Facebook的Litho;Google的Flutter
## 小知识点
   Flutter把Skia引擎直接集成到App中，就像一个游戏App一样
