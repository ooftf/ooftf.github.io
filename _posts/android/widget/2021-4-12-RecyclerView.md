---
layout: post
author: "ooftf"
tags: Android
---

### ScrollView嵌套RecyclerView自动滚动

[相关文章](https://blog.csdn.net/beibaokongming/article/details/79581232)

- 解决方案一： ScrollView的子viewGroup中增加android:descendantFocusability="blocksDescendants" 属性
- 解决方案二： ScrollView的直接子view 上添加focusableInTouchMode=true

### ScrollView 嵌套RecyclerView + GridLayoutManager 只显示一行或者不显示，需要滑动RecyclerView
解决方案：将 ScrollView 替换为 NestScrollView
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


## DiffUtil
```kotlin
   val diffResult = DiffUtil.calculateDiff(object : DiffUtil.Callback() {
            override fun getOldListSize(): Int {
            }

            override fun getNewListSize(): Int {
            }

            override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
                // 数据的ID是否相同
            }

            override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
                // 数据的内容是否相同
            }

        }, true)
        diffResult.dispatchUpdatesTo(recyclerView.adapter)
```
# RecyclerView 源码分析
## 分析RecyclerView.onLayout
```java
@Override
protected void RecyclerView.onLayout(boolean changed, int l, int t, int r, int b) {
    dispatchLayout();
    mFirstLayoutComplete = true;
}

void RecyclerView.dispatchLayout() {
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
        mLayout.setExactMeasureSpecsFrom(this);
        dispatchLayoutStep2();
    } else {
        // always make sure we sync them (to ensure mode is exact)
        mLayout.setExactMeasureSpecsFrom(this);
    }
    dispatchLayoutStep3();
}
```

### RecyclerView.dispatchLayoutStep1
* 只有 State.STEP_START 状态才会执行  dispatchLayoutStep1
* 调用 dispatchLayoutStep1 会将mState.mLayoutStep 置为 State.STEP_LAYOUT;
* 与 RecyclerView 动画相关
    - RunSimpleAnimations 动画只会计算属性，不会进行布局，
    - RunPredictiveAnimations 会通过 LayoutManager.onLayoutChildren 进行空间布局


### RecyclerView.dispatchLayoutStep3
* 将 mState.mLayoutStep 重新置为 State.STEP_START
* 添加动画：mViewInfoStore.addToPostLayout(holder, animationInfo);
* 执行动画：mViewInfoStore.process(mViewInfoProcessCallback);
*  mLayout.removeAndRecycleScrapInt(mRecycler);  将 Recycler.mAttachedScrap 中没有复用的 ViewHolder 添加到  RecycledViewPool 中
*  清除 Recycler.mChangedScrap 中的缓存
*  mRecycler.updateViewCacheSize(); 将 Recycler.mCachedViews 中的没有复用的 ViewHolder 添加到 RecycledViewPool 中  


### RecyclerView.dispatchLayoutStep2()
```java
private void dispatchLayoutStep2() {
    mState.mInPreLayout = false;
    mLayout.onLayoutChildren(mRecycler, mState);
    mState.mRunSimpleAnimations = mState.mRunSimpleAnimations &&mItemAnimator != null;
    mState.mLayoutStep = State.STEP_ANIMATIONS;
}
```
## LinearLayoutManager.onLayoutChildren

在每次向RecyclerView填充表项之前都会先清空 LayoutManager 中现存表项，将它们 detach 并同时缓存入 mAttachedScrap列表中。
```java
public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {

    ...
    detachAndScrapAttachedViews(recycler);
    ...
    fill(recycler, mLayoutState, state, false);
    ...
}
```
 
    LayoutManager.detachAndScrapAttachedViews  
        LayoutManager.scrapOrRecycleView   
            LayoutManager.detachViewAt(index)   
                LayoutManager.detachViewInternal   
                    ChildHelper.detachViewFromParent   
                         mCallback.detachViewFromParent(offset);   
                            RecyclerView.initChildrenHelper()  
                                RecyclerView.this.detachViewFromParent(offset);  
                                    ViewGroup.detachViewFromParent  
            Recycler.scrapView 
                mAttachedScrap.add(holder);


```java
private void scrapOrRecycleView(Recycler recycler, int index, View view) {
    final ViewHolder viewHolder = getChildViewHolderInt(view);
    if (viewHolder.shouldIgnore()) {
        if (DEBUG) {
            Log.d(TAG, "ignoring view " + viewHolder);
        }
        return;
    }
    if (viewHolder.isInvalid() && !viewHolder.isRemoved()
            && !mRecyclerView.mAdapter.hasStableIds()) {
        //notifyDataSetChanged 会走这个分支
        // 将viewHolder 缓存到 mCachedViews 和 RecycledViewPool
        removeViewAt(index);
        recycler.recycleViewHolderInternal(viewHolder);
    } else {
        //初次布局
        detachViewAt(index);
        recycler.scrapView(view);
        mRecyclerView.mViewInfoStore.onViewDetached(viewHolder);
    }
}
```
```java
void Recycler.recycleViewHolderInternal(ViewHolder holder) {
    ...
    //如果已经超过 mCachedViews 最大容量 按找 FIFO 规则将 ViewHodler 转移到 RecycledViewPool
    if (cachedViewSize >= mViewCacheMax && cachedViewSize > 0) {
                   recycleCachedViewAt(0);
                   cachedViewSize--;
               }
    mCachedViews.add(targetCacheIndex, holder);
    ...
}
```

#### LinearLayoutManager.fill 

```
LinearLayoutManager.fill 
    LinearLayoutManager.layoutChunk
        LayoutState.next 
            Recycler.getViewForPosition
                Recycler.tryGetViewHolderForPositionByDeadline
```

```java
 ViewHolder tryGetViewHolderForPositionByDeadline(int position,
            boolean dryRun, long deadlineNs) {
    // 通过 position 从 mChangedScrap 中获取ViewHolder
    holder = getChangedScrapViewForPosition(position);
    // 通过 position 从 mAttachedScrap 或者 mCachedViews 中获取 ViewHolder
    holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun)
    // 通过ItemId 从 mAttachedScrap 或者 mCachedViews 中获取 ViewHolder
    holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition)
    // 通过 position 和 type 从 【用户自定义缓存ViewCacheExtension】中获取ViewHolder
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
    } else if (!holder.isBound() || holder.needsUpdate() || holderisInvalid()) {
        final int offsetPosition = mAdapterHelper.findPositionOffse(position);
        bound = tryBindViewHolderByDeadline(holder, offsetPosition,position, deadlineNs);
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

## ViewGroup.removeView 和 ViewGroup.detachViewFromParent 区别
* ViewGroup.removeView 和 ViewGroup.detachViewFromParent 功能相似，都会调用ViewGroup.removeFromArray将子控件从父控件的孩子列表中移除
* detachViewFromParent 更轻量，仅仅调用 ViewGroup.removeFromArray 将子控件从父控件的孩子列表中移除
* ViewGroup.removeView 会关注动画、焦点、触摸等事件，并且会重新布局和重新绘画。最终也会调用ViewGroup.removeFromArray 将子控件从父控件的孩子列表中移除


## RecyclerView.onMeasuer
```java
protected void onMeasure(int widthSpec, int heightSpec) {
    final int widthMode = MeasureSpec.getMode(widthSpec);
    final int heightMode = MeasureSpec.getMode(heightSpec);
    mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
    mLastAutoMeasureSkippedDueToExact =
            widthMode == MeasureSpec.EXACTLY && heightMode ==MeasureSpecEXACTLY;
    if (mLastAutoMeasureSkippedDueToExact || mAdapter == null) {
        return;
    }
    // 上面代码就是正常View的 onMeasure  当不是精确测量并且Adapter不null的候，开始按照 LayoutManager 的方式测量；如果是精确测量，是不执行下面代码的
    //首先调用 dispatchLayoutStep1 对Children进行布局
    if (mState.mLayoutStep == State.STEP_START) {
        dispatchLayoutStep1();
    }
    mLayout.setMeasureSpecs(widthSpec, heightSpec);
    mState.mIsMeasuring = true;
    dispatchLayoutStep2();
    mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
    mLastAutoMeasureNonExactMeasuredWidth = getMeasuredWidth();
    mLastAutoMeasureNonExactMeasuredHeight = getMeasuredHeight();
}
```

### LinearLayoutManager.generateDefaultLayoutParams

当ItemView没有设置 LayoutParams 的时候，就会调用这个方法添加 LayoutParams


## ViewInfoStore
ViewInfoStore 记录 RecyclerView 动画相关信息，通过 process 执行动画
```java
void addToPreLayout(RecyclerView.ViewHolder holder, RecyclerViewItemAnimator.ItemHolderInfo info) {
    InfoRecord record = mLayoutHolderMap.get(holder);
    if (record == null) {
        record = InfoRecord.obtain();
        mLayoutHolderMap.put(holder, record);
    }
    record.preInfo = info;
    record.flags |= FLAG_PRE;
}
void addToPostLayout(RecyclerView.ViewHolder holder, RecyclerViewItemAnimator.ItemHolderInfo info) {
    InfoRecord record = mLayoutHolderMap.get(holder);
    if (record == null) {
        record = InfoRecord.obtain();
        mLayoutHolderMap.put(holder, record);
    }
    record.postInfo = info;
    record.flags |= FLAG_POST;
}

```

## 缓存总结
RecycerView 中用来管理缓存的类是 Recycler ，缓存相关逻辑都在 Recycler 中实现
* mChangedScrap  
  - ArrayList<ViewHolder>
  - 缓存屏幕内的ViewHolder
* mAttachedScrap 
  - ArrayList<ViewHolder> 
  - 缓存屏幕内的ViewHolder
* mCachedViews 
  - ArrayList<ViewHolder>
  - 缓存屏幕外的 ViewHolder
  - 默认容量为 2，如果达到最大值，会按照 FIFO 规则，调用 Recycler.recycleCachedViewAt 方法将 ViewHodler 转移到  RecycledViewPool
* mViewCacheExtension 
  - 自定义的缓存
* RecycledViewPool 
  - 数据结构
    ```java
    SparseArray<ScrapData> mScrap = new SparseArray<>();
    ScrapData.mScrapHeap  = new ArrayList<ViewHolder> 
    ```
  - 添加到RecyclerdViewPool ViewHolder 会清除状态信息
    ```java
       void resetInternal() {
            mFlags = 0;
            mPosition = NO_POSITION;
            mOldPosition = NO_POSITION;
            mItemId = NO_ID;
            mPreLayoutPosition = NO_POSITION;
            mIsRecyclableCount = 0;
            mShadowedHolder = null;
            mShadowingHolder = null;
            clearPayload();
            mWasImportantForAccessibilityBeforeHidden = ViewCompat.IMPORTANT_FOR_ACCESSIBILITY_AUTO;
            mPendingAccessibilityState = PENDING_ACCESSIBILITY_STATE_NOT_SET;
            clearNestedRecyclerViewIfNotNested(this);
        }
    ```

#### adapter.notify
* notifyItemChanged 会将发生改变的放入到 mChangedScrap ;没有改变的放入到mAttachedScrap 中
* notifyDataSetChanged 会将所有 ViewHolder 通过 recycler.recycleViewHolderInternal 加入到 mCachedViews 和 RecycledViewPool 中
* notifyItemRemoved 会将所有 ViewHolder 缓存到 mAttachedScrap 中
* 查看 RecyclerViewDataObserver 可知事件变化监听都是通过调用 requestLayout 触发重新布局来实现的(也就是会触发 回收所有View 重新布局)，不同的是：不同的事件会将不同的 ViewHolder 置为不同的 FLAG 因此会添加到不同的缓存中
#### 相关知识点
* 缓存 viewHolder 的入口为 LayoutManager.scrapOrRecycleView
* 放入 mAttachedScrap 和 mAttachedScrap 时，移除View 的方式是 detachView；而放入 mCachedViews 和 RecycledViewPool 时，移除 View 的方式是 removeView
* adapter.notifyDataSetChanged 会先调用 processDataSetCompletelyChanged 将 ViewHolder 置为ViewHolder.FLAG_UPDATE、ViewHolder.FLAG_INVALID  然后通过 requestLayout 间接调用LayoutManager.scrapOrRecycleView 调用 Recycler.recycleViewHolderInternal 将 ViewHolder 添加到 mCachedViews 或者 RecycledViewPool
* dispatchLayoutStep3 调用 removeAndRecycleScrapInt 将  mAttachedScrap 中的 ViewHodler  通过 quickRecycleScrapView 调用 recycleViewHolderInternal 方法转移到 mCachedViews 或者  RecycledViewPool 中；并且将  mChangedScrap 清除；所以dispatchLayoutStep3执行完毕后，mAttachedScrap和mChangedScrap被清空； mCachedViews 和 RecycledViewPool 有缓存
* mCachedViews 和 RecycledViewPool 不同的是 mCachedViews 还保留了位置信息，可以通过相同的 position 复用但是要重新 bind ,RecycledViewPool 已经变为一个"空白"ViewHodler
* mAttachedScrap 和 mCachedViews 的区别是 mAttachedScrap 获取到的 ViewHolder 不需要重新 bind；
* mChangedScrap 和 mAttachedScrap 的区别是  mChangedScrap 需要重新 bindview ;mAttachedScrap 不需要重新bind

### 实战探索
#### LinearLayoutManager & wrap_content
* onMeasure dispatchLayoutStep1 dispatchLayoutStep2 onLayoutChildren
* onMeasure dispatchLayoutStep2 onLayoutChildren
* onMeasure dispatchLayoutStep2 onLayoutChildren
* onMeasure dispatchLayoutStep2 onLayoutChildren
* onMeasure dispatchLayoutStep2 onLayoutChildren
* onMeasure dispatchLayoutStep2 onLayoutChildren
* onLayout dispatchLayout  dispatchLayoutStep3

#### LinearLayoutManager & match_parent
* onMeasure
* onMeasure
* onMeasure
* onMeasure
* onMeasure
* onMeasure
* onLayout dispatchLayoutStep1 dispatchLayoutStep2 onLayoutChildren * dispatchLayoutStep3



## 优化方向

* onCreateViewHolder 创建View使用代码创建对象的方式，而不是 xml 省去解析 xml 的时间
* 如果是多列布局比如 GridLayoutManager 可以设置最大缓存个数避免滑动过程中调用 onCreateViewHolder 
  - recyclerView.recycledViewPool.setMaxRecycledViews()  设置 RecyclerViewPool 指定ViewType 大小
  - recyclerView.setItemViewCacheSize 设置 mCachedViews 大小
* 使用 DiffUtil 优化更新
* 使用 DataBinding 可以在不使用 adapter.nofity 的情况下改变 ItemView 的子 View 的内容
* RecyclerView 尽可能使用精确大小比如 match_parent 而不是 wrap_content 可见减少 LayoutManager.onLayoutChildren 的调用次数


## ViewHolder 的 FLAG  
有助于理解 ViewHolder 的状态，不是很重要
```java
 /**
 * This ViewHolder has been bound to a position; mPosition, mItemIand mItemViewType
 * are all valid.
 */
static final int FLAG_BOUND = 1 << 0
/**
 * The data this ViewHolder's view reflects is stale and needs to brebound
 * by the adapter. mPosition and mItemId are consistent.
 */
static final int FLAG_UPDATE = 1 << 1
/**
 * This ViewHolder's data is invalid. The identity implied bmPosition and mItemId
 * are not to be trusted and may no longer match the item view type.
 * This ViewHolder must be fully rebound to different data.
 */
static final int FLAG_INVALID = 1 << 2
/**
 * This ViewHolder points at data that represents an item previouslremoved from the
 * data set. Its view may still be used for things like outgoinanimations.
 */
static final int FLAG_REMOVED = 1 << 3
/**
 * This ViewHolder should not be recycled. This flag is set visetIsRecyclable()
 * and is intended to keep views around during animations.
 */
static final int FLAG_NOT_RECYCLABLE = 1 << 4
/**
 * This ViewHolder is returned from scrap which means we arexpecting an addView call
 * for this itemView. When returned from scrap, ViewHolder stays ithe scrap list until
 * the end of the layout pass and then recycled by RecyclerView if iis not added back to
 * the RecyclerView.
 */
static final int FLAG_RETURNED_FROM_SCRAP = 1 << 5
/**
 * This ViewHolder is fully managed by the LayoutManager. We do noscrap, recycle or remove
 * it unless LayoutManager is replaced.
 * It is still fully visible to the LayoutManager.
 */
static final int FLAG_IGNORE = 1 << 7
/**
 * When the View is detached form the parent, we set this flag sthat we can take correct
 * action when we need to remove it or add it back.
 */
static final int FLAG_TMP_DETACHED = 1 << 8
/**
 * Set when we can no longer determine the adapter position of thiViewHolder until it is
 * rebound to a new position. It is different than FLAG_INVALIbecause FLAG_INVALID is
 * set even when the type does not match. AlsoFLAG_ADAPTER_POSITION_UNKNOWN is set as soon
 * as adapter notification arrives vs FLAG_INVALID is set lazilbefore layout is
 * re-calculated.
 */
static final int FLAG_ADAPTER_POSITION_UNKNOWN = 1 << 9
/**
 * Set when a addChangePayload(null) is called
 */
static final int FLAG_ADAPTER_FULLUPDATE = 1 << 10
/**
 * Used by ItemAnimator when a ViewHolder's position changes
 */
static final int FLAG_MOVED = 1 << 11
/**
 * Used by ItemAnimator when a ViewHolder appears in pre-layout
 */
static final int FLAG_APPEARED_IN_PRE_LAYOUT = 1 << 12;
```

