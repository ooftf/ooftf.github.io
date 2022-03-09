---
layout: post
author: "ooftf"
tags: Android
title: StaggeredGridLayoutManager
top: true
---

# StaggeredGridLayoutManager 瀑布流头部空白和 item 漂移问题

## 什么情况下会出现移位问题

![](https://ooftf-blog-image.oss-cn-beijing.aliyuncs.com/img/1.png)


如果当前布局情况如图所示左上角出现了一片空白区域，StaggeredGridLayoutManager  会根据当前的 GapStrategy 采取不同的处理

1. GapStrategy 默认为 GAP_HANDLING_MOVE_ITEMS_BETWEEN_SPANS   这种情况在 RecyclerView 滚动状态变为 SCROLL_STATE_IDLE 也就是不滚动状态时，会检查是否有空白如果有空白就会对布局通过动画进行调整
2. GapStrategy 设置为 GAP_HANDLING_NONE 时，StaggeredGridLayoutManager 不会自动调整布局。如果想要触发调整布局需要主动调用 StaggeredGridLayoutManager.invalidateSpanAssignments 这时候就会导致 item 出现位移


## 为什么会出现左上角出现空白的情况呢
首先明确一点当触发重新布局的时候，LayoutManager 会优先保证正在显示的控件位置不变，然后以正在展示的内容为基础，向上向下挨个布局。
1. item  onBindViewHolder 后的第一次 onMeasure 并不是 item 的最终大小。比如异步图片加载、Cube 开启 Falcon_memory_optimize 后的异步渲染。


![](https://ooftf-blog-image.oss-cn-beijing.aliyuncs.com/img/2.png)

如图所示第一个表示正常布局情况，第二个表示向上划动一段距离，item2 大小发生了变化就成了第三张图的样子，这时候再划动到顶部，左上角就会出现空白。

2. notifyDataSetChange 和 invalidateSpanAssignments 导致界面重布局。

![](https://ooftf-blog-image.oss-cn-beijing.aliyuncs.com/img/3.png)

如图所示当 图一 向上划动一段距离后变成 图二，这个时候调用了 notifyDataSetChange 或者  invalidateSpanAssignments 导致 StaggeredGridLayoutManager 保证可见 item （3，4，5，6，7，8，9）不变的情况下向上向下布局，因为是从下向上布局所以 item2 会比 item1 更早布局，因此就会生成图三的布局。再向下划动到顶部就会变成图四的样子，左上角出现空白。这种情况多出现在第一行左边高度大于右边高度的情况。

## 如何完美解决瀑布流布局问题
1. 保证 item  onBindViewHolder 后的第一次 onMeasure 就是 item 的最终大小
2. 当加载下一页等需要添加 item 的动作时，不要调用 notifyDataSetChange ，而是 notifyItemInserted 等局部更新方法。

* 像 Glide 等异步加载图片框架必然会导致 item 大小改变，可以采用服务器下发图片大小的方式，在 onBindViewHolder 直接根据图片大小将控件设置为最终大小。
* 如果 item 必然会发生大小改变。那么无论怎么设置都会出现空白或者位移问题，只是出现的时机和表现形式不同罢了。


## 实操解析
### 第一种配置
默认配置，即 GapStrategy 的值为 StaggeredGridLayoutManager.GAP_HANDLING_MOVE_ITEMS_BETWEEN_SPANS
### 第二种配置
```kotlin
staggeredGridLayoutManager.gapStrategy = StaggeredGridLayoutManager.GAP_HANDLING_NONE

recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
    override fun onScrollStateChanged(recyclerView: RecyclerView, newState: Int) {
        staggeredGridLayoutManager.invalidateSpanAssignments()
    }
})
```

### 这两种配置有什么不同吗：
首先要明确：这两种配置都不能解决 item 大小延迟改变导致的 item 漂移问题，只是发生偏移的时机和表现形式不同

#### 第一点不同
* 第一种配置：偏移发生在停止滚动的时候，漂移是通过动画来完成的。
* 第二种配置：偏移发生在 onScrollStateChanged ，漂移时没有动画。

#### 第二点不同
* 第一种配置，如果 item 大小不会发生改变是不会出现漂移现象的。
* 第二种配置，在 item 大小不会发生改变的情况下也会出现漂移现象，具体原因可参考第三张图。

因为第一种配置在 滚动状态变为 SCROLL_STATE_IDLE 时会先调用 StaggeredGridLayoutManager.hasGapsToFix 检查是否需要重新布局，StaggeredGridLayoutManager.hasGapsToFix 的大概逻辑是判断 nextChild 的 top 要大于 child top，否则就有可能 hasGaps （当然还有一些其他的判断，比如是否 fullSpan、是否是从上到下布局，暂时就不做详细解析了，有兴趣的可以直接查看源码）。在 hasGapsToFix 返回为 false 的情况下是不会触发重布局的。而第二种配置，会直接调用重布局即使 item 大小没有发生变化。