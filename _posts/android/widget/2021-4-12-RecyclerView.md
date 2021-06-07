---
layout: post
author: "ooftf"
tags: Android
---

### ScrollView嵌套RecyclerView自动滚动

![相关文章](https://blog.csdn.net/beibaokongming/article/details/79581232)

- 解决方案一： ScrollView的子viewGroup中增加android:descendantFocusability="blocksDescendants" 属性
- 解决方案二： ScrollView的直接子view 上添加focusableInTouchMode=true
### ScrollView 嵌套RecyclerView + GridLayoutManager 只显示一行或者不显示，需要滑动RecyclerView
    将ScrollView换位NestScrollView
### RecyclerView 中EditText singleLine模式下，作为最后一个Item，输入换行崩溃，某些机型（华为）
```
java.lang.IllegalStateException: focus search returned a view that wasn't able to take focus!
    at android.widget.TextView.onKeyUp(TextView.java:8643)
    at android.widget.EditText.onKeyUp(EditText.java:289)
    at android.view.KeyEvent.dispatch(KeyEvent.java:3094)
    at android.view.View.dispatchKeyEvent(View.java:13537)
    at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1973)
    at com.ooftf.layout.kv.KvLayout.dispatchKeyEvent(KvLayout.kt:423)
    at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1973)
    at android.view.ViewGroup.dispatchKeyEvent(ViewGroup.java:1973)
    at com.android.internal.policy.DecorView.superDispatchKeyEvent(DecorView.java:668)
    at com.android.internal.policy.PhoneWindow.superDispatchKeyEvent(PhoneWindow.java:1993)
    at android.app.Activity.dispatchKeyEvent(Activity.java:4107)
    at androidx.core.app.ComponentActivity.superDispatchKeyEvent(ComponentActivity.java:122)
    at androidx.core.view.KeyEventDispatcher.dispatchKeyEvent(KeyEventDispatcher.java:84)
    at androidx.core.app.ComponentActivity.dispatchKeyEvent(ComponentActivity.java:140)
    at androidx.appcompat.app.AppCompatActivity.dispatchKeyEvent(AppCompatActivity.java:569)
    at androidx.appcompat.view.WindowCallbackWrapper.dispatchKeyEvent(WindowCallbackWrapper.java:59)
    at androidx.appcompat.app.AppCompatDelegateImpl$AppCompatWindowCallback.dispatchKeyEvent(AppCompatDelegateImpl.java
    at com.baidu.mobstat.ak.dispatchKeyEvent(SourceFile:50)
    at com.baidu.mobstat.ak.dispatchKeyEvent(SourceFile:50)
    at com.android.internal.policy.DecorView.dispatchKeyEvent(DecorView.java:516)
    at android.view.ViewRootImpl$ViewPostImeInputStage.processKeyEvent(ViewRootImpl.java:6038)
    at android.view.ViewRootImpl$ViewPostImeInputStage.onProcess(ViewRootImpl.java:5906)
    at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:5350)
    at android.view.ViewRootImpl$InputStage.onDeliverToNext(ViewRootImpl.java:5403)
    at android.view.ViewRootImpl$InputStage.forward(ViewRootImpl.java:5369)
    at android.view.ViewRootImpl$AsyncInputStage.forward(ViewRootImpl.java:5527)
    at android.view.ViewRootImpl$InputStage.apply(ViewRootImpl.java:5377)
    at android.view.ViewRootImpl$AsyncInputStage.apply(ViewRootImpl.java:5584)
    at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:5350)
    at android.view.ViewRootImpl$InputStage.onDeliverToNext(ViewRootImpl.java:5403)
    at android.view.ViewRootImpl$InputStage.forward(ViewRootImpl.java:5369)
    at android.view.ViewRootImpl$InputStage.apply(ViewRootImpl.java:5377)
    at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:5350)
    at android.view.ViewRootImpl.deliverInputEvent(ViewRootImpl.java:8325)
    at android.view.ViewRootImpl.doProcessInputEvents(ViewRootImpl.java:8245)
    at android.view.ViewRootImpl.enqueueInputEvent(ViewRootImpl.java:8198)
    at android.view.ViewRootImpl$ViewRootHandler.handleMessage(ViewRootImpl.java:5111)
    at android.os.Handler.dispatchMessage(Handler.java:110)
    at android.os.Looper.loop(Looper.java:219)
    at android.app.ActivityThread.main(ActivityThread.java:8387)
    at java.lang.reflect.Method.invoke(Native Method)
    at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:513)
    at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1055)
```
* 方案一
```
  override fun dispatchKeyEvent(event: KeyEvent?): Boolean {
        //解决EditText作为RecyclerView的最后一行时，输入换行，在某些机型（华为）崩溃的问题
        try {
            return super.dispatchKeyEvent(event)
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return true

    }
```
* 方案二
   把EditText的android:imeOptions="actionNone" 即可
### RecyclerView clear再添加数据，界面有闪烁
    是因为RecyclerView有默认动画， itemAnimator = null 可去除默认动画


# RecyclerView 源码分析
## onLayout
```java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    dispatchLayout();
    mFirstLayoutComplete = true;
}

void dispatchLayout() {
    if (mAdapter == null) {
        Log.w(TAG, "No adapter attached; skipping layout");
        // leave the state in START
        return;
    }
    if (mLayout == null) {
        Log.e(TAG, "No layout manager attached; skipping layout");
        // leave the state in START
        return;
    }
    mState.mIsMeasuring = false;
    // If the last time we measured children in onMeasure, we skipped the measurement and layout
    // of RV children because the MeasureSpec in both dimensions was EXACTLY, and current
    // dimensions of the RV are not equal to the last measured dimensions of RV, we need to
    // measure and layout children one last time.
    boolean needsRemeasureDueToExactSkip = mLastAutoMeasureSkippedDueToExact
                    && (mLastAutoMeasureNonExactMeasuredWidth != getWidth()
                    || mLastAutoMeasureNonExactMeasuredHeight != getHeight());
    mLastAutoMeasureNonExactMeasuredWidth = 0;
    mLastAutoMeasureNonExactMeasuredHeight = 0;
    mLastAutoMeasureSkippedDueToExact = false;
    if (mState.mLayoutStep == State.STEP_START) {
        dispatchLayoutStep1();
        mLayout.setExactMeasureSpecsFrom(this);
        dispatchLayoutStep2();
    } else if (mAdapterHelper.hasUpdates()
            || needsRemeasureDueToExactSkip
            || mLayout.getWidth() != getWidth()
            || mLayout.getHeight() != getHeight()) {
        // First 2 steps are done in onMeasure but looks like we have to run again due to
        // changed size.
        // TODO(shepshapard): Worth a note that I believe
        //  "mLayout.getWidth() != getWidth() || mLayout.getHeight() != getHeight()" above is
        //  not actually correct, causes unnecessary work to be done, and should be
        //  removed. Removing causes many tests to fail and I didn't have the time to
        //  investigate. Just a note for the a future reader or bug fixer.
        mLayout.setExactMeasureSpecsFrom(this);
        dispatchLayoutStep2();
    } else {
        // always make sure we sync them (to ensure mode is exact)
        mLayout.setExactMeasureSpecsFrom(this);
    }
    dispatchLayoutStep3();
}
```
```java
    protected void onMeasure(int widthSpec, int heightSpec) {
            final int widthMode = MeasureSpec.getMode(widthSpec);
            final int heightMode = MeasureSpec.getMode(heightSpec);
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
            mLastAutoMeasureSkippedDueToExact =
                    widthMode == MeasureSpec.EXACTLY && heightMode == MeasureSpec.EXACTLY;
            if (mLastAutoMeasureSkippedDueToExact || mAdapter == null) {
                return;
            }

            // 上面代码就是正常View的 onMeasure  当不是精确测量并且Adapter不为null的时候，开始按照 LayoutManager 的方式测量

            //首先调用 dispatchLayoutStep1 对Children进行布局
            if (mState.mLayoutStep == State.STEP_START) {
                dispatchLayoutStep1();
            }
            // set dimensions in 2nd step. Pre-layout should happen with old dimensions for
            // consistency
            mLayout.setMeasureSpecs(widthSpec, heightSpec);
            mState.mIsMeasuring = true;
            dispatchLayoutStep2();

            // now we can get the width and height from the children.
            mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);

            // if RecyclerView has non-exact width and height and if there is at least one child
            // which also has non-exact width & height, we have to re-measure.
            if (mLayout.shouldMeasureTwice()) {
                mLayout.setMeasureSpecs(
                        MeasureSpec.makeMeasureSpec(getMeasuredWidth(), MeasureSpec.EXACTLY),
                        MeasureSpec.makeMeasureSpec(getMeasuredHeight(), MeasureSpec.EXACTLY));
                mState.mIsMeasuring = true;
                dispatchLayoutStep2();
                // now we can get the width and height from the children.
                mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
            }

            mLastAutoMeasureNonExactMeasuredWidth = getMeasuredWidth();
            mLastAutoMeasureNonExactMeasuredHeight = getMeasuredHeight();
    }
```
* dispatchLayoutStep1
  会触发LayoutManger.onLayoutChildren
  The first step of a layout where we;
  - process adapter updates
  - decide which animation should run
  - save information about current views
  - If necessary, run predictive layout and save its information
* dispatchLayoutStep2
    会触发LayoutManger.onLayoutChildren
    The second layout step where we do the actual layout of the views for the final state.
    This step might be run multiple times if necessary (e.g. measure).
* dispatchLayoutStep3
    The final step of the layout where we save the information about views for animations,
    trigger animations and do any necessary cleanup.
  

### LinearLayoutManager.generateDefaultLayoutParams
## LinearLayoutManager.onLayoutChildren

在每次向RecyclerView填充表项之前都会先清空 LayoutManager 中现存表项，将它们 detach 并同时缓存入 mAttachedScrap列表中。
```java
public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {

    ...
    detachAndScrapAttachedViews(recycler);
    ...
    fill(recycler, mLayoutState, state, false);
    ...

```
1. LayoutManager.detachAndScrapAttachedViews $\rightarrow$ LayoutManager.scrapOrRecycleView $\rightarrow$
   1. LayoutManager.detachViewAt(index) $\rightarrow$ LayoutManager.detachViewInternal $\rightarrow$ ChildHelper.detachViewFromParent $\rightarrow$ mCallback.detachViewFromParent(offset); $\uparrow$ RecyclerView.initChildrenHelper() $\rightarrow$ RecyclerView.this.detachViewFromParent(offset); $\Leftrightarrow$ ViewGroup.detachViewFromParent
   2. Recycler.scrapView $\rightarrow$ mAttachedScrap.add(holder);
2. LinearLayoutManager.fill $\rightarrow$ LinearLayoutManager.layoutChunk $\rightarrow$ LayoutState.next $\rightarrow$ Recycler.getViewForPosition $\rightarrow$ Recycler.tryGetViewHolderForPositionByDeadline
    ```java
     ViewHolder tryGetViewHolderForPositionByDeadline(int position,
                boolean dryRun, long deadlineNs) {
        // 通过 position 从 mChangedScrap 中获取ViewHolder
        holder = getChangedScrapViewForPosition(position);
        // 通过 position 从 mAttachedScrap 中获取 ViewHolder
        holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun)
        // 通过ItemId 从 mAttachedScrap 中获取 ViewHolder
        holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition)
        // 通过 position 和 type 从 【用户自定义缓存ViewCacheExtension】中获取 ViewHolder
        final View view = mViewCacheExtension
                            .getViewForPositionAndType(this, position, type);
        holder = getChildViewHolder(view);
        // 从 RecycledViewPool.mScrap 中获取 ViewHolder
        holder = getRecycledViewPool().getRecycledView(type);

        holder = mAdapter.createViewHolder(RecyclerView.this, type)

        // 判断是否需要重新调用 adapter.bindViewHolder
        boolean bound = false;
        if (mState.isPreLayout() && holder.isBound()) {
            holder.mPreLayoutPosition = position;
        } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {
            final int offsetPosition = mAdapterHelper.findPositionOffset(position);
            bound = tryBindViewHolderByDeadline(holder, offsetPosition, position, deadlineNs);
        }

        // 设置 itemView 的 Layoutparams 
        final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
        final LayoutParams rvLayoutParams;
        if (lp == null) {
            rvLayoutParams = (LayoutParams) generateDefaultLayoutParams();
            holder.itemView.setLayoutParams(rvLayoutParams);
        } else if (!checkLayoutParams(lp)) {
            rvLayoutParams = (LayoutParams) generateLayoutParams(lp);
            holder.itemView.setLayoutParams(rvLayoutParams);
        } else {
            rvLayoutParams = (LayoutParams) lp;
        }
    }
    ```
## ViewGroup.removeView 和 ViewGroup.detachViewFromParent
* ViewGroup.removeView 和 ViewGroup.detachViewFromParent 功能相似，都会调用ViewGroup.removeFromArray将子控件从父控件的孩子列表中移除
* detachViewFromParent 更轻量，仅仅调用 ViewGroup.removeFromArray 将子控件从父控件的孩子列表中移除
* ViewGroup.removeView 会关注动画、焦点、触摸等事件，并且会重新布局和重新绘画。最终也会调用ViewGroup.removeFromArray 将子控件从父控件的孩子列表中移除

## post-layout 和 pre-layout
# RecyclerView 为了实现表项动画，进行了 2 次布局