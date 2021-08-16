---
title: DataBinding
---


### 优点
* 有效避免空指针异常
* 代码变得更少了
* google 提供的 adapter 属性 太少

### 缺点
* 异常排查变得复杂
* View 复用性降低



## 原理解析

[原理解析](https://blog.csdn.net/sted_zxz/article/details/79593575) 
### DataBinding 配置
```groovy
android{
    buildFeatures {
        dataBinding true
    }
}
```
#### 相关类解释 
1. DataBinderMapper
2. {name}Binding
3. {name}BindingImpl


#### binding 相关方法
* invalidateAll()
* mDirtyFlags
  这是一个标志位，用来判断变量有没有改变，其运作方式如下
  初始值为mDirtyFlags = 0xffffffffffffffffL; 转换为二进制为 64个1 表示所有变量都有改变
  当mDirtyFlags = 0的时候代表所有变量都没有改变，不需要给View重新赋值

  例如：变量name的标志位是第三位  
  如果只检测到name值改变，就会将mDirtyFlags 与 0x4 （100）进行或运算，其目的就是将第三位置为1  
  在判断是否改变的时候，将 与 0xc（1100）进行与运算，如果结果不为0表示 name字段有改变，需要调用  BindingAdapter重新赋值
#### 问
* DataBinding 生成的 Binding 类在哪里
  \build\generated\data_binding_base_class_source_out\debug\out\{applicationId}\databinding\{name}Binding
* DataBinding 生成的 BindingImpl 类在哪里
  \build\generated\ap_generated_sources\debug\out\{applicationId}\databinding\{name}BindingImpl

## be careful
* java.lang.NoClassDefFoundError: Failed resolution of: Landroidx/databinding/DataBinderMapperImpl;
  即使 App Module 没有使用 dataBinding 也要配置 dataBinding,如果使用了 kotlin ，还需要添加 katp 插件
  ```groovy
  buildFeatures {
          dataBinding true
  }
  ```

## 源码分析
```java
    public void setTextObservable(@Nullable androidx.databinding.ObservableField<java.lang.String> TextObservable) {
        updateRegistration(0, TextObservable);
        this.mTextObservable = TextObservable;
        synchronized(this) {
            mDirtyFlags |= 0x1L;
        }
        notifyPropertyChanged(BR.textObservable);
        super.requestRebind();
    }
    public void setTextLiveData(@Nullable androidx.lifecycle.MutableLiveData<java.lang.String> TextLiveData) {
        updateLiveDataRegistration(1, TextLiveData);
        this.mTextLiveData = TextLiveData;
        synchronized(this) {
            mDirtyFlags |= 0x2L;
        }
        notifyPropertyChanged(BR.textLiveData);
        super.requestRebind();
    }
    public void setViewModel(@Nullable com.ooftf.demo.databinding.MainViewModel ViewModel) {
        this.mViewModel = ViewModel;
        synchronized(this) {
            mDirtyFlags |= 0x10L;
        }
        notifyPropertyChanged(BR.viewModel);
        super.requestRebind();
    }
    public void setText(@Nullable java.lang.String Text) {
        this.mText = Text;
        synchronized(this) {
            mDirtyFlags |= 0x20L;
        }
        notifyPropertyChanged(BR.text);
        super.requestRebind();
    }
```

分析:
-------------
会根据三种不同类型（Observable、LiveData、其他）做不同的实现，如果是 Observable 会调用 updateRegistration 注册监听，如果是 LiveData 会调用 updateLiveDataRegistration 注册监听

将新的值赋给全局变量

将 mDirtyFlags 对应的标志位，置为 “改变”

调用 notifyPropertyChanged 通知对应 id 改变

调用 requestRebind

接下来先分析 requestRebind

```java
    protected void requestRebind() {
        if (mContainingBinding != null) {
            mContainingBinding.requestRebind();
        } else {
            final LifecycleOwner owner = this.mLifecycleOwner;
            if (owner != null) {
                Lifecycle.State state = owner.getLifecycle().getCurrentState();
                if (!state.isAtLeast(Lifecycle.State.STARTED)) {
                    return; // wait until lifecycle owner is started
                }
            }
            synchronized (this) {
                if (mPendingRebind) {
                    return;
                }
                mPendingRebind = true;
            }
            if (USE_CHOREOGRAPHER) {
                mChoreographer.postFrameCallback(mFrameCallback);
            } else {
                mUIThreadHandler.post(mRebindRunnable);
            }
        }
    }
```
分析
----
判断当前状态是否是可见的，如果不可见直接 return

如果状态都符合，采用 Choreographer 或者 Handler 向主线程发送回调,最终都会执行到 mRebindRunnable.run

接下来看 mRebindRunnable.run

```java

    /**
     * Runnable executed on animation heartbeat to rebind the dirty Views.
     */
    private final Runnable mRebindRunnable = new Runnable() {
        @Override
        public void run() {
            synchronized (this) {
                mPendingRebind = false;
            }
            processReferenceQueue();
            if (VERSION.SDK_INT >= VERSION_CODES.KITKAT) {
                // Nested so that we don't get a lint warning in IntelliJ
                if (!mRoot.isAttachedToWindow()) {
                    // Don't execute the pending bindings until the View
                    // is attached again.
                    mRoot.removeOnAttachStateChangeListener(ROOT_REATTACHED_LISTENER);
                    mRoot.addOnAttachStateChangeListener(ROOT_REATTACHED_LISTENER);
                    return;
                }
            }
            executePendingBindings();
        }
    };
```
分析
----
判断是否已经添加到 window 上，如果没有添加 OnAttachStateChangeListener ，然后直接 return

如果 isAttachedToWindow 返回 true 执行 executePendingBindings 方法

```java
    public void executePendingBindings() {
        if (mContainingBinding == null) {
            executeBindingsInternal();
        } else {
            mContainingBinding.executePendingBindings();
        }
    }

    // 接着看 executeBindingsInternal 方法

    private void executeBindingsInternal() {
        if (mIsExecutingPendingBindings) {
            requestRebind();
            return;
        }
        if (!hasPendingBindings()) {
            return;
        }
        mIsExecutingPendingBindings = true;
        mRebindHalted = false;
        if (mRebindCallbacks != null) {
            mRebindCallbacks.notifyCallbacks(this, REBIND, null);

            if (mRebindHalted) {
                mRebindCallbacks.notifyCallbacks(this, HALTED, null);
            }
        }
        if (!mRebindHalted) {
            executeBindings();
            if (mRebindCallbacks != null) {
                mRebindCallbacks.notifyCallbacks(this, REBOUND, null);
            }
        }
        mIsExecutingPendingBindings = false;
    }
```
```java
    @Override
    protected void executeBindings() {
        long dirtyFlags = 0;
        synchronized(this) {
            dirtyFlags = mDirtyFlags;
            mDirtyFlags = 0;
        }
        java.lang.String textLiveDataGetValue = null;
        androidx.databinding.ObservableField<java.lang.String> viewModelTextObservable = null;
        androidx.lifecycle.MutableLiveData<java.lang.String> viewModelTextLiveData = null;
        java.lang.String viewModelTextLiveDataGetValue = null;
        java.lang.String text = mText;
        java.lang.String viewModelText = null;
        java.lang.String viewModelTextObservableGet = null;
        androidx.databinding.ObservableField<java.lang.String> textObservable = mTextObservable;
        androidx.lifecycle.MutableLiveData<java.lang.String> textLiveData = mTextLiveData;
        com.ooftf.demo.databinding.MainViewModel viewModel = mViewModel;
        java.lang.String textObservableGet = null;

        if ((dirtyFlags & 0x50L) != 0) {
        }
        if ((dirtyFlags & 0x44L) != 0) {



                if (textObservable != null) {
                    // read textObservable.get()
                    textObservableGet = textObservable.get();
                }
        }
        if ((dirtyFlags & 0x48L) != 0) {



                if (textLiveData != null) {
                    // read textLiveData.getValue()
                    textLiveDataGetValue = textLiveData.getValue();
                }
        }
        if ((dirtyFlags & 0x63L) != 0) {


            if ((dirtyFlags & 0x61L) != 0) {

                    if (viewModel != null) {
                        // read viewModel.textObservable
                        viewModelTextObservable = viewModel.getTextObservable();
                    }
                    updateRegistration(0, viewModelTextObservable);


                    if (viewModelTextObservable != null) {
                        // read viewModel.textObservable.get()
                        viewModelTextObservableGet = viewModelTextObservable.get();
                    }
            }
            if ((dirtyFlags & 0x62L) != 0) {

                    if (viewModel != null) {
                        // read viewModel.textLiveData
                        viewModelTextLiveData = viewModel.getTextLiveData();
                    }
                    updateLiveDataRegistration(1, viewModelTextLiveData);


                    if (viewModelTextLiveData != null) {
                        // read viewModel.textLiveData.getValue()
                        viewModelTextLiveDataGetValue = viewModelTextLiveData.getValue();
                    }
            }
            if ((dirtyFlags & 0x60L) != 0) {

                    if (viewModel != null) {
                        // read viewModel.text
                        viewModelText = viewModel.getText();
                    }
            }
        }
        // batch finished
        if ((dirtyFlags & 0x50L) != 0) {
            // api target 1

            androidx.databinding.adapters.TextViewBindingAdapter.setText(this.button, text);
        }
        if ((dirtyFlags & 0x40L) != 0) {
            // api target 1

            androidx.databinding.adapters.TextViewBindingAdapter.setTextWatcher(this.button, (androidx.databinding.adapters.TextViewBindingAdapter.BeforeTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.OnTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.AfterTextChanged)null, buttonandroidTextAttrChanged);
            androidx.databinding.adapters.TextViewBindingAdapter.setTextWatcher(this.mboundView2, (androidx.databinding.adapters.TextViewBindingAdapter.BeforeTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.OnTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.AfterTextChanged)null, mboundView2androidTextAttrChanged);
            androidx.databinding.adapters.TextViewBindingAdapter.setTextWatcher(this.mboundView3, (androidx.databinding.adapters.TextViewBindingAdapter.BeforeTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.OnTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.AfterTextChanged)null, mboundView3androidTextAttrChanged);
            androidx.databinding.adapters.TextViewBindingAdapter.setTextWatcher(this.mboundView4, (androidx.databinding.adapters.TextViewBindingAdapter.BeforeTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.OnTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.AfterTextChanged)null, mboundView4androidTextAttrChanged);
            androidx.databinding.adapters.TextViewBindingAdapter.setTextWatcher(this.mboundView5, (androidx.databinding.adapters.TextViewBindingAdapter.BeforeTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.OnTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.AfterTextChanged)null, mboundView5androidTextAttrChanged);
            androidx.databinding.adapters.TextViewBindingAdapter.setTextWatcher(this.mboundView6, (androidx.databinding.adapters.TextViewBindingAdapter.BeforeTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.OnTextChanged)null, (androidx.databinding.adapters.TextViewBindingAdapter.AfterTextChanged)null, mboundView6androidTextAttrChanged);
        }
        if ((dirtyFlags & 0x44L) != 0) {
            // api target 1

            androidx.databinding.adapters.TextViewBindingAdapter.setText(this.mboundView2, textObservableGet);
        }
        if ((dirtyFlags & 0x48L) != 0) {
            // api target 1

            androidx.databinding.adapters.TextViewBindingAdapter.setText(this.mboundView3, textLiveDataGetValue);
        }
        if ((dirtyFlags & 0x60L) != 0) {
            // api target 1

            androidx.databinding.adapters.TextViewBindingAdapter.setText(this.mboundView4, viewModelText);
        }
        if ((dirtyFlags & 0x61L) != 0) {
            // api target 1

            androidx.databinding.adapters.TextViewBindingAdapter.setText(this.mboundView5, viewModelTextObservableGet);
        }
        if ((dirtyFlags & 0x62L) != 0) {
            // api target 1

            androidx.databinding.adapters.TextViewBindingAdapter.setText(this.mboundView6, viewModelTextLiveDataGetValue);
        }
    }

```

根据 dirtyFlags 判断数据是否改变，如果数据发生改变，在判断是否是可观察类型，如果是可观察类型，调用 updateRegistration 更新监听；然后获取到改变后的数据
调用对应的 BindingAdapter 设置给控件 

我们再回过头来看一下 updateRegistration 的源码

```java
    /**
     * @hide
     */
    protected boolean updateRegistration(int localFieldId, Observable observable) {
        return updateRegistration(localFieldId, observable, CREATE_PROPERTY_LISTENER);
    }

    /**
     * @hide
     */
    protected boolean updateRegistration(int localFieldId, ObservableList observable) {
        return updateRegistration(localFieldId, observable, CREATE_LIST_LISTENER);
    }

    /**
     * @hide
     */
    protected boolean updateRegistration(int localFieldId, ObservableMap observable) {
        return updateRegistration(localFieldId, observable, CREATE_MAP_LISTENER);
    }

    /**
     * @hide
     */
    protected boolean updateLiveDataRegistration(int localFieldId, LiveData<?> observable) {
        mInLiveDataRegisterObserver = true;
        try {
            return updateRegistration(localFieldId, observable, CREATE_LIVE_DATA_LISTENER);
        } finally {
            mInLiveDataRegisterObserver = false;
        }
    }
```

从代码可知，针对四种不同的被观察对象(Observable ObservableList ObservableMap LiveData)，分别有四种不同的 CREATE_XXXX_LISTENER 负责对应的监听，下面我们以 CREATE_LIVE_DATA_LISTENER 为例，进行分析

```java
    private static final CreateWeakListener CREATE_LIVE_DATA_LISTENER = new CreateWeakListener() {
        @Override
        public WeakListener create(
                ViewDataBinding viewDataBinding,
                int localFieldId,
                ReferenceQueue<ViewDataBinding> referenceQueue
        ) {
            return new LiveDataListener(viewDataBinding, localFieldId, referenceQueue)
                    .getListener();
        }
    };
```

```java
    private static class LiveDataListener implements Observer,
            ObservableReference<LiveData<?>> {
        final WeakListener<LiveData<?>> mListener;
        @Nullable
        WeakReference<LifecycleOwner> mLifecycleOwnerRef = null;

        public LiveDataListener(
                ViewDataBinding binder,
                int localFieldId,
                ReferenceQueue<ViewDataBinding> referenceQueue
        ) {
            mListener = new WeakListener(binder, localFieldId, this, referenceQueue);
        }

        @Nullable
        private LifecycleOwner getLifecycleOwner() {
            WeakReference<LifecycleOwner> ownerRef = this.mLifecycleOwnerRef;
            if (ownerRef == null) {
                return null;
            }
            return ownerRef.get();
        }

        @Override
        public void setLifecycleOwner(@Nullable LifecycleOwner lifecycleOwner) {
            LifecycleOwner previousOwner = getLifecycleOwner();
            LifecycleOwner newOwner = lifecycleOwner;
            LiveData<?> liveData = mListener.getTarget();
            if (liveData != null) {
                if (previousOwner != null) {
                    liveData.removeObserver(this);
                }
                if (newOwner != null) {
                    liveData.observe(newOwner, this);
                }
            }
            if (newOwner != null) {
                mLifecycleOwnerRef = new WeakReference<LifecycleOwner>(newOwner);
            }
        }

        @Override
        public WeakListener<LiveData<?>> getListener() {
            return mListener;
        }

        @Override
        public void addListener(LiveData<?> target) {
            LifecycleOwner lifecycleOwner = getLifecycleOwner();
            if (lifecycleOwner != null) {
                target.observe(lifecycleOwner, this);
            }
        }

        @Override
        public void removeListener(LiveData<?> target) {
            target.removeObserver(this);
        }

        @Override
        public void onChanged(@Nullable Object o) {
            ViewDataBinding binder = mListener.getBinder();
            if (binder != null) {
                binder.handleFieldChange(mListener.mLocalFieldId, mListener.getTarget(), 0);
            }
        }
    }
```

分析：

通过 target.observer 监听数据变化，当 onChanged 获取到数据变化之后再通知给 ViewDataBinding.handleFieldChange 字段发生改变 ，接下来查看 handleFiledChnage 方法

```java
    protected void handleFieldChange(int mLocalFieldId, Object object, int fieldId) {
        if (mInLiveDataRegisterObserver || mInStateFlowRegisterObserver) {
            return;
        }
        boolean result = onFieldChange(mLocalFieldId, object, fieldId);
        if (result) {
            requestRebind();
        }
    }

    @Override
    protected boolean onFieldChange(int localFieldId, Object object, int fieldId) {
        switch (localFieldId) {
            case 0 :
                return onChangeViewModelTextObservable((androidx.databinding.ObservableField<java.lang.String>) object, fieldId);
            case 1 :
                return onChangeViewModelTextLiveData((androidx.lifecycle.MutableLiveData<java.lang.String>) object, fieldId);
            case 2 :
                return onChangeTextObservable((androidx.databinding.ObservableField<java.lang.String>) object, fieldId);
            case 3 :
                return onChangeTextLiveData((androidx.lifecycle.MutableLiveData<java.lang.String>) object, fieldId);
        }
        return false;
    }


    private boolean onChangeTextLiveData(androidx.lifecycle.MutableLiveData<java.lang.String> TextLiveData, int fieldId) {
        if (fieldId == BR._all) {
            synchronized(this) {
                    mDirtyFlags |= 0x8L;
            }
            return true;
        }
        return false;
    }
```
分析：

handleFieldChange 先调用 onFieldChange 再调用 onChangeTextLiveData 将 mDirtyFlags 中对应的 bit 位 置为已改变，然后调用 requestRebind 通知界面数据改变。requestRebind 上面已经做了分析，就不做重复分析了

