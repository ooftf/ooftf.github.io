---
layout: post
author: "ooftf"
tags: Android
---
![Fragment生命周期](https://upload-images.jianshu.io/upload_images/1688279-0424d62f50035b43.png?imageMogr2/auto-orient/strip|imageView2/2/w/317/format/webp)
### 为什么会出现fragment重叠现象:
    因为activity内存回收的时候，fragment被FragmentManager保存了下来，当再次创建Activity的时候，原来被保存下来的fragment默认为显示状态
    解决方式：
1.    fragment保存显隐状态
2.    activity重新创建的时候通过 tag找到原来fragment并重新设置显隐状态

### FragmentPagerAdapter和FragmentStatePagerAdapter
* FragmentPagerAdapter内fragment当移除视野外之后不会重新创建，会保存在内存中
* FragmentStatePagerAdapter 当fragment移除视野之外就会销毁

### fragmentTransaction.add(containerViewId, fragment, tagId).commitAllowingStateLoss()
并不会立刻将 frament 添加到 FragmentManager 中，所以在这之后调用FragmentManager.findFragmentByTag 是找不到fragment。走过一个Handler周期就可以通过FragmentManager.findFragmentByTag获取到了。具体解决方法可以参考 Glide 添加 SupportRequestManagerFragment 的方式


## ViewPager2 + Fragment
#### FragmentStateAdapter
* 从 0 Fragment 开始向左滑动一点 就会开始创建 1 Fragment 生命周期会走到 onStart,并不会执行 onResume。只有继续向左滑动当前显示Fragment 变为 1 的时候，才会执行  1  的 onResume 方法，并且 0 Fragment执行 onPaused 方法
* 多次左右切换并不会重新创建 Fragment 只会执行 onResume 和 onPaused 方法

## ViewPager + Fragment

## Fragment + BottomBar
* Fragment 使用 hide 和 show 的方式控制显隐
* Fragment的生命周期和 Activity 生命周期同步。
* 左右切换的时候会调用 FragmentManager.hideFragment 和 FragmentManager.showFragment 改变 Fragment.mHidden 的值，但是不会执行 onResume onPaused onStart onStop 等生命周期方法

## 理解 Fragment 的生命周期
FragmentManager
```java
void moveToState(@NonNull Fragment f, int newState) {
        FragmentStateManager fragmentStateManager = mFragmentStore.getFragmentStateManager(f.mWho);
        if (fragmentStateManager == null) {
            fragmentStateManager = new FragmentStateManager(mLifecycleCallbacksDispatcher, f);
            fragmentStateManager.setFragmentManagerState(Fragment.CREATED);
        }
        newState = Math.min(newState, fragmentStateManager.computeMaxState());
        if (f.mState <= newState) {

            switch (f.mState) {
                case Fragment.INITIALIZING:
                    if (newState > Fragment.INITIALIZING) {
                        if (isLoggingEnabled(Log.DEBUG)) Log.d(TAG, "moveto ATTACHED: " + f);

                        fragmentStateManager.attach(mHost, this, mParent);
                    }
                    // fall through
                case Fragment.ATTACHED:
                    if (newState > Fragment.ATTACHED) {
                        fragmentStateManager.create();
                    }
                    // fall through
                case Fragment.CREATED:
    
                    if (newState > Fragment.CREATED) {
                        fragmentStateManager.createView(mContainer);
                        fragmentStateManager.activityCreated();
                        fragmentStateManager.restoreViewState();
                    }
                    // fall through
                case Fragment.ACTIVITY_CREATED:
                    if (newState > Fragment.ACTIVITY_CREATED) {
                        fragmentStateManager.start();
                    }
                    // fall through
                case Fragment.STARTED:
                    if (newState > Fragment.STARTED) {
                        fragmentStateManager.resume();
                    }
            }
        } else if (f.mState > newState) {
            switch (f.mState) {
                case Fragment.RESUMED:
                    if (newState < Fragment.RESUMED) {
                        fragmentStateManager.pause();
                    }
                    // fall through
                case Fragment.STARTED:
                    if (newState < Fragment.STARTED) {
                        fragmentStateManager.stop();
                    }
                    // fall through
                case Fragment.ACTIVITY_CREATED:
                    if (newState < Fragment.ACTIVITY_CREATED) {
                       
                        FragmentAnim.AnimationOrAnimator anim = null;
                        if (f.mView != null && f.mContainer != null) {
                            // Stop any current animations:
                            f.mContainer.endViewTransition(f.mView);
                            f.mView.clearAnimation();
            
                            if (!f.isRemovingParent()) {
                   
                                ViewGroup container = f.mContainer;
                                View view = f.mView;
                               
                                container.removeView(view);
                               
                                if (container != f.mContainer) {
                                    return;
                                }
                            }
                        }
                        destroyFragmentView(f);
                       
                    }
                    // fall through
                case Fragment.CREATED:
                    if (newState < Fragment.CREATED) {
                       fragmentStateManager.destroy(mHost, mNonConfig);
                    }
                    // fall through
                case Fragment.ATTACHED:
                    if (newState < Fragment.ATTACHED) {
                        fragmentStateManager.detach(mNonConfig);
                    }
            }
        }fff
    }
```