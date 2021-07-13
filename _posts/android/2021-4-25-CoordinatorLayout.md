---
title: CoordinatorLayout
---

## CoordinatorLayout
* 默认情况下 child 布局类似 FrameLayout
* 关键属性  app:layout_behavior="@string/appbar_scrolling_view_behavior" 决定了。Child 之间的相对位置，和对于滑动事件的消费优先级

## AppBarLayout
* 继承自 LinearLayout ，所以 Child 布局和 LinearLayout 相似
* android:layout_height="200dp" 决定了 AppBarLayout 的最大显示高度
* 最小显示高度等于 (noScroll child 高度) + (exitUntilCollapsed child minHight)  
* AppBarLayout 滚动并不是控件大小改变了，而是滚出了屏幕

#### AppBarLayout Child 关键属性 app:layout_scrollFlag
* noScroll
  * 禁用视图上的滚动。此标志不应与任何其他滚动标志结合使用。
* scroll
  * 视图将与滚动事件直接相关。需要设置此标志才能使任何其他标志生效。如果在此之前的任何同级视图没有此标志，则此值无效
* exitUntilCollapsed
  * 退出（滚动屏幕）时，视图将滚动，直到它“折叠”。折叠高度由视图的最小高度定义。 （滚动到最小高度，不再滚动）
* enterAlways
  * 当进入（在屏幕上滚动）视图将在任何向下滚动事件上滚动，无论滚动视图是否也在滚动。这通常被称为“快速回报”模式。
* enterAlwaysCollapsed
  * 'enterAlways' 的附加标志，它修改返回视图以仅最初滚动回其折叠高度。一旦滚动视图到达其滚动范围的末尾，该视图的其余部分将被滚动到视图中。（enterAlways 情况下，并不是显示全部，而是 minHeight，当正常 enter 才会慢慢显示全部）
* snap
  * 在滚动结束时，如果视图只是部分可见，那么它将被捕捉并滚动到它最近的边缘。
* snapMargins
  * 与 'snap' 一起使用的附加标志。如果设置，视图将捕捉到其顶部和底部边距，而不是视图本身的边缘。


![AppbarLayout](https://raw.githubusercontent.com/ooftf/Material/master/img/blog/AppbarLayout.png)

## CollapsingToolbarLayout
* 它需要放在AppBarLayout布局里面，并且作为AppBarLayout的直接子View。
#### app:layout_collapseMode
#### app:contentScrim




[推荐文章](https://www.jianshu.com/p/bbc703a0015e)