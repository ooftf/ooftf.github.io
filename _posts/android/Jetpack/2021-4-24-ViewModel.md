---
layout: post
author: "ooftf"
top: true
tags: [Android,ViewModel]
title: ViewModel
---
### 项目配置ViewModel支持
```groovy
    def lifecycle_version = "2.3.1"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
```

如果要使用 ViewModelScope 需要添加

```groovy
implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0'
```

### 为什么横竖屏切换后，Activity是新对象，ViewModel还是原来的对象
* 由代码可知我们获取ViewModel 是通过 ViewModelProvider.get 方法获取的
  ```kotlin
  ViewModelProvider(this).get(MyViewModel::class.java)
  ```
* 查看内部实现可知 ViewModel 是通过 mViewModelStore.get(key) 方法获取的，此处Key为固定值 "androidx.lifecycle.ViewModelProvider.DefaultKey"
  ```java
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        ViewModel viewModel = mViewModelStore.get(key);

        if (modelClass.isInstance(viewModel)) {
            if (mFactory instanceof OnRequeryFactory) {
                ((OnRequeryFactory) mFactory).onRequery(viewModel);
            }
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }
        if (mFactory instanceof KeyedFactory) {
            viewModel = ((KeyedFactory) mFactory).create(key, modelClass);
        } else {
            viewModel = mFactory.create(modelClass);
        }
        mViewModelStore.put(key, viewModel);
        return (T) viewModel;
    }
  ```
* 此处的mViewModelStore是我们创建 ViewModelProvider 传入的 ViewModelStoreOwner 通过 ViewModelStoreOwner.getViewModelStore()获取到的
  ```java
  public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
      this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
              ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
              : NewInstanceFactory.getInstance());
  }
  ```
* ComponentActivity.getViewModelStore() 是在Activity的父类 androidx.activity.ComponentActivity 内实现的
  ```java
    @NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        if (getApplication() == null) {
            throw new IllegalStateException("Your activity is not yet attached to the "
                    + "Application instance. You can't request ViewModel before onCreate call.");
        }
        if (mViewModelStore == null) {
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                // Restore the ViewModelStore from NonConfigurationInstances
                mViewModelStore = nc.viewModelStore;
            }
            if (mViewModelStore == null) {
                mViewModelStore = new ViewModelStore();
            }
        }
        return mViewModelStore;
    }
  ```
* 通过学习横竖屏切换相关的知识可知：我横竖屏切换时可以通过 onRetainNonConfigurationInstance 方法返回需要保存的对象，然后在新的Activity中通过 getLastNonConfigurationInstance 方法 获取到保存的对象  

* 查看 ComponentActivity.onRetainNonConfigurationInstance 方法可知，在横竖屏切换时保存了viewModelStore对象。因此新旧Activity获取到的viewModelStore是同一个实例，自然获取到的ViewModel也是同一个实例
  ```java
    @Override
    @Nullable
    public final Object ComponentActivity.onRetainNonConfigurationInstance() {
        Object custom = onRetainCustomNonConfigurationInstance();

        ViewModelStore viewModelStore = mViewModelStore;
        if (viewModelStore == null) {
            // No one called getViewModelStore(), so see if there was an existing
            // ViewModelStore from our last NonConfigurationInstance
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                viewModelStore = nc.viewModelStore;
            }
        }

        if (viewModelStore == null && custom == null) {
            return null;
        }

        NonConfigurationInstances nci = new NonConfigurationInstances();
        nci.custom = custom;
        nci.viewModelStore = viewModelStore;
        return nci;
    }
  ``` 

### ViweModel.clear 的调用时机

```java
public ComponentActivity() {
    ...
    getLifecycle().addObserver(new LifecycleEventObserver() {
        @Override
        public void onStateChanged(@NonNull LifecycleOwner source,
                @NonNull Lifecycle.Event event) {
            if (event == Lifecycle.Event.ON_DESTROY) {
                if (!isChangingConfigurations()) {
                    getViewModelStore().clear();
                }
            }
        }
    });
    ...
}
```

由代码可知 Activity在创建的时候通过Lifecycle 监听ON_DESTROY生命周期，通过isChangingConfigurations判断如果不是横竖屏切换，就会执行clear方法
