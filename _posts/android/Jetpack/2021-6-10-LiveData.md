---
layout: post
author: "ooftf"
title: LiveData
top: true
tags: [Android,LiveData]
---

## LiveData 原理分析
### LiveData.post() 
```java
protected void postValue(T value) {
    boolean postTask;
    synchronized (mDataLock) {
        postTask = mPendingData == NOT_SET;//判断是否需要发送更新 value 的事件 mPostValueRunnable ，在执行 mPostValueRunable 后就会将 mPendingData 设置为 NOT_SET
        mPendingData = value;  // mPendingData 是触发 mPostValueRunnable 要赋给 value 的值，这里就是更新 mPendingData 的值
    }
    if (!postTask) {   // 如果连续 postValue 两个值，只会postToMainThread 一次 mPostValueRunnable 事件，mPendingData 会是最后一次 postValue 的值
        return;
    }
    ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);  // 向主线程发送更新事件
}
```
------------------
```java
private final Runnable mPostValueRunnable = new Runnable() {
    @Override
    public void run() {
        Object newValue;
        synchronized (mDataLock) {
            newValue = mPendingData;  
            mPendingData = NOT_SET;   
        }
        // 总的来说就是拿到 postValue 方法传入的最新值,将 mPendingData 置为 NOT_SET 表示没有更新事件 mPostValueRunnable，并调用 setValue
        setValue((T) newValue);
    }
};
```
### LiveData.setValue() 
```java
@MainThread
protected void setValue(T value) {
    assertMainThread("setValue"); // 判断当前线程，setValue 只能在主线程中执行
    mVersion++; // 增加 LiveData 的版本号
    mData = value;  //更新 LiveData 的数据为最新之
    dispatchingValue(null); // 通知订阅者
}
```

```java
    // 这个方法的作用是通知订阅者数据发生了改变，有两种情况，
    //一种是仅通知指定单个 ObserverWrapper 这种情况用于刚订阅或者订阅者的声明周期发生改变的时候
    //第二种情况是通知所有的 ObserverWrapper 
    void dispatchingValue(@Nullable ObserverWrapper initiator) { 
        
        if (mDispatchingValue) {
            mDispatchInvalidated = true;
            return;
        }
        mDispatchingValue = true;
        do {
            mDispatchInvalidated = false;
            if (initiator != null) {
                considerNotify(initiator);
                initiator = null;
            } else {
                for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                        mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                    considerNotify(iterator.next().getValue());
                    if (mDispatchInvalidated) {
                        break;
                    }
                }
            }
        } while (mDispatchInvalidated);
        mDispatchingValue = false;
    }
```
```java
 private void considerNotify(ObserverWrapper observer) {
        if (!observer.mActive) {   // observer.mActive 表示当前 ObserverWrapper 是否是活跃状态， mActive = true 表示 Lifecycle.Event 大于 START
            return;
        }
        // 为了防止漏掉事件的情况，再判断一次 shouldBeActive
        if (!observer.shouldBeActive()) {
            observer.activeStateChanged(false);
            return;
        }
        if (observer.mLastVersion >= mVersion) { // 如果ObserverWrapper当前版本大于等于 LiveData 版本，表示 ObserverWrapper 上次收到的事件已经是最新的，不需要重新通知observer更新
            return;
        }
        observer.mLastVersion = mVersion; // 将ObserverWrapper赋值为最新版本，并且通知 observer 更新

        observer.mObserver.onChanged((T) mData);
    }
```
### LiveData.observe() 
```java
@MainThread
public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
    assertMainThread("observe");
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {
        // 如果 Lifecycle 已经处于 DESTROYED 状态 没必要再订阅通知，直接return
        return;
    }

    // 对 observer 做一层生命周期封装，通过监听 owner 的生命周期变化，
    // 在 owner 变为 START 状态时，通过 onStateChanged 调用 dispatchingValue 通知 observer.onChange
    // 在 owner 改变状态时，更新 ObserverWrapper.mActive

    LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
    ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper); // 如果当前 observer 已经存在，会重新设置 owner，如果owner也相同则会 抛出异常
    if (existing != null && !existing.isAttachedTo(owner)) {
        throw new IllegalArgumentException("Cannot add the same observer"
                + " with different lifecycles");
    }
    if (existing != null) {
        return;
    }
    owner.getLifecycle().addObserver(wrapper);//wrapper 实现了 LifecycleEventObserver 接口，监听生命周期，生命周期变化会在 onStateChanged 方法中回调
}
```
LifecycleBoundObserver.onStateChanged
```java
@Override
public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
    if (mOwner.getLifecycle().getCurrentState() == DESTROYED) { //如果已经处于销毁状态，直接移除监听
        removeObserver(mObserver);
        return;
    }

    activeStateChanged(shouldBeActive());// 设置活跃状态变化
}
```
```java
// 判断是否是活跃状态
@Override
boolean shouldBeActive() {
    return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
}
```
```java
void activeStateChanged(boolean newActive) {
    if (newActive == mActive) {
        return;
    }
    // immediately set active state, so we'd never dispatch anything to inactive
    // owner
    mActive = newActive;
    boolean wasInactive = LiveData.this.mActiveCount == 0; // mActiveCount是LiveData所有订阅者中处于活跃状态的的订阅者数目；wasInactive 是指原来 LiveData 是否都处于非活跃状态
    LiveData.this.mActiveCount += mActive ? 1 : -1;
    if (wasInactive && mActive) {//如果原来  LiveData 处于非活跃状态 并且 新的状态为活跃，调用 LiveData.onActive 方法
        onActive();
    }
    if (LiveData.this.mActiveCount == 0 && !mActive) { //如果满足 （LiveData.this.mActiveCount == 0），通过 LiveData.this.mActiveCount += mActive ? 1 : -1; 可知 LiveData.this.mActiveCount=1 即 LiveData 原来处于活跃状态，现在处于非活跃状态，调用 LiveData.onInactive
        onInactive();
    }
    if (mActive) { //如果是活跃状态，通知次订阅者数值发生改变
        dispatchingValue(this);
    }
}
```