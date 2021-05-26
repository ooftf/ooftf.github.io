---
layout: post
author: "ooftf"
tags: Android
---
![Fragment生命周期](https://upload-images.jianshu.io/upload_images/1688279-0424d62f50035b43.png?imageMogr2/auto-orient/strip|imageView2/2/w/317/format/webp)
### 为什么出出现fragment重叠现象:
    因为activity内存回收的时候，fragment被FragmentManager保存了下来，当再次创建Activity的时候，原来被保存下来的fragment默认为显示状态
    解决方式：
1.    fragment保存显隐状态
2.    activity重新创建的时候通过 tag找到原来fragment并重新设置显隐状态


### FragmentPagerAdapter和FragmentStatePagerAdapter
    FragmentPagerAdapter内fragment当移除视野外之后不会重新创建，会保存在内存中
    FragmentStatePagerAdapter 当fragment移除视野之外就会销毁
### fragmentTransaction.add(containerViewId, fragment, tagId).commitAllowingStateLoss()
并不会立刻将 frament 添加到 FragmentManager 中，所以在这之后调用FragmentManager.findFragmentByTag 是找不到fragment。走过一个Handler周期就可以通过FragmentManager.findFragmentByTag获取到了。具体解决方法可以参考 Glide 添加 SupportRequestManagerFragment 的方式