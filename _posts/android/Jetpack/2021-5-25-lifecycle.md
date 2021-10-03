

## 配置

```groovy
dependencies {
        def lifecycle_version = "2.4.0-beta01"
        def arch_version = "2.1.0"

        // ViewModel    对 ViewModel 扩展了 viewModelScope
        implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
        // LiveData
        implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
        // Lifecycles only (without ViewModel or LiveData)
        implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version"

        // Saved state module for ViewModel
        implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:$lifecycle_version"

        // Annotation processor
        kapt "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
        // alternately - if using Java8, use the following instead of lifecycle-compiler
        implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

        // optional - helpers for implementing LifecycleOwner in a Service
        implementation "androidx.lifecycle:lifecycle-service:$lifecycle_version"

        // optional - ProcessLifecycleOwner provides a lifecycle for the whole application process
        implementation "androidx.lifecycle:lifecycle-process:$lifecycle_version"

        // optional - ReactiveStreams support for LiveData
        implementation "androidx.lifecycle:lifecycle-reactivestreams-ktx:$lifecycle_version"

        // optional - Test helpers for LiveData
        testImplementation "androidx.arch.core:core-testing:$arch_version"
    }
```
## 自定义 LifecycleOwner
```java
public class MainActivity extends Activity implements LifecycleOwner {
    private LifecycleRegistry mLifecycleRegistry;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mLifecycleRegistry = new LifecycleRegistry(this);
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);
        mLifecycleRegistry.addObserver(new TestObserver());
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }

    @Override
    public void onStart() {
        super.onStart();
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_START);
    }

    @Override
    public void onResume() {
        super.onResume();
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_RESUME);
    }

    @Override
    public void onPause() {
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_PAUSE);
        super.onPause();
    }

    @Override
    public void onStop() {
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_STOP);
        super.onStop();
    }

    @Override
    public void onDestroy() {
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_DESTROY);
        super.onDestroy();
    }
}
```

## 源码分析
```java
@Override
    public void LifecycleRegistry.addObserver(@NonNull LifecycleObserver observer) {
        State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
        ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
        ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);

        if (previous != null) {
            return;
        }
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            // it is null we should be destroyed. Fallback quickly
            return;
        }

        boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent;
        State targetState = calculateTargetState(observer);
        mAddingObserverCounter++;
        while ((statefulObserver.mState.compareTo(targetState) < 0
                && mObserverMap.contains(observer))) {
            pushParentState(statefulObserver.mState);
            statefulObserver.dispatchEvent(lifecycleOwner, upEvent(statefulObserver.mState));
            popParentState();
            // mState / subling may have been changed recalculate
            targetState = calculateTargetState(observer);
        }

        if (!isReentrance) {
            // we do sync only on the top level.
            sync();
        }
        mAddingObserverCounter--;
    }
```

## ProcessLifecycle
监听App前后台切换的事件

引入

```groovy
implementation "androidx.lifecycle:lifecycle-process:$lifecycle_version"
```
ProcessLifecycle 使用的是 startup 方式 进行初始化所以不需要编写任何初始化相关的代码

使用：
```kotlin
        ProcessLifecycleOwner.get().lifecycle.addObserver(object :DefaultLifecycleObserver{
            override fun onStart(owner: LifecycleOwner) {
                // 从后台切换到前台
            }

            override fun onStop(owner: LifecycleOwner) {
                // 从前台切换到后台
            }
        })
```
在 ProcessLifecycle 中 onStart 和 onResume 相同，onStop 和 onPause 相同

## lifecycle 是如何感知组件的声明周期的
TODO


## 为什么使用 @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)  就可以监听到生命周期⌚️

OnLifecycleEvent 已经标记为 Deprecated，官方解释：应避免使用生成代码或者反射，推荐使用 DefaultLifecycleObserver 或者 LifecycleEventObserver


分两种情况处理

1. 不是匿名内部类，并且依赖了 lifecycle-compiler ，在这种情况下会为对应的监听 XXXXObserver 生成一个辅助类 XXXXObserver__LifecycleAdapter
2. 如果是匿名内部类或者没有依赖 lifecycle-compiler 就会用反射的方式活去信息生成 ReflectiveGenericLifecycleObserver 对象

源码分析 ：

```java
    /**
    这个方法的作用就是，根据 Observer 的类型不同用不同的方式解析成 LifecycleEventObserver，分为四种情况
    1. 本身就是 LifecycleEventObserver 的实例，直接返回就可以了
    2. 是 FullLifecycleObserver 的实例，返回 FullLifecycleObserverAdapter 做一层 LifecycleEventObserver 到 FullLifecycleObserver 的映射
    3. 使用注解的方式 & 不是匿名内部类 & 生成了对应的 adapter ，这种方式使用反射生成 adapter 的实例并返回
    4. 使用注解 &（匿名内部类｜没有生成 adapter），这种方式会返回 ReflectiveGenericLifecycleObserver ，使用反射获取 OnLifecycleEvent 注解值和方法的映射关系
    */
    @NonNull
    @SuppressWarnings("deprecation")
    static LifecycleEventObserver Lifecycling.lifecycleEventObserver(Object object) {
        boolean isLifecycleEventObserver = object instanceof LifecycleEventObserver;
        boolean isFullLifecycleObserver = object instanceof FullLifecycleObserver;
        if (isLifecycleEventObserver && isFullLifecycleObserver) {
            return new FullLifecycleObserverAdapter((FullLifecycleObserver) object,
                    (LifecycleEventObserver) object);
        }
        if (isFullLifecycleObserver) {
            return new FullLifecycleObserverAdapter((FullLifecycleObserver) object, null);
        }

        if (isLifecycleEventObserver) {
            return (LifecycleEventObserver) object;
        }

        // 上面两种情况都是 Observer 直接就是 LifecycleEventObserver 或者 FullLifecycleObserver 的子类，直接返回 LifecycleEventObserver 就可以了
        final Class<?> klass = object.getClass();
        // 获取通过 getObserverConstructorType 获取 Observer 的类型是第三种还是第四种，下面有 getObserverConstructorType 的分析
        int type = getObserverConstructorType(klass);
        if (type == GENERATED_CALLBACK) {
            List<Constructor<? extends GeneratedAdapter>> constructors =
                    sClassToAdapters.get(klass);
            if (constructors.size() == 1) {
                GeneratedAdapter generatedAdapter = createGeneratedAdapter(
                        constructors.get(0), object);
                return new SingleGeneratedAdapterObserver(generatedAdapter);
            }
            GeneratedAdapter[] adapters = new GeneratedAdapter[constructors.size()];
            for (int i = 0; i < constructors.size(); i++) {
                adapters[i] = createGeneratedAdapter(constructors.get(i), object);
            }
            return new CompositeGeneratedAdaptersObserver(adapters);
        }
        return new ReflectiveGenericLifecycleObserver(object);
    }
```

```java
    /**
    主要就是调用一下 resolveObserverCallbackType 
    */
    private static int getObserverConstructorType(Class<?> klass) {
        Integer callbackCache = sCallbackCache.get(klass);
        if (callbackCache != null) {
            return callbackCache;
        }
        int type = resolveObserverCallbackType(klass);
        sCallbackCache.put(klass, type);
        return type;
    }


    private static int resolveObserverCallbackType(Class<?> klass) {
        // anonymous class bug:35073837
        if (klass.getCanonicalName() == null) {
            return REFLECTIVE_CALLBACK;
        }
        // 获取 Observer 对应的 Adapter 的构造方法，如果获取到的构造方法不为 null 表示存在生成代码 adapter ，返回 GENERATED_CALLBACK
        Constructor<? extends GeneratedAdapter> constructor = generatedConstructor(klass);
        if (constructor != null) {
            sClassToAdapters.put(klass, Collections
                    .<Constructor<? extends GeneratedAdapter>>singletonList(constructor));
            return GENERATED_CALLBACK;
        }
        // 下面的逻辑是处理反射这种情况的
        @SuppressWarnings("deprecation")
        boolean hasLifecycleMethods = ClassesInfoCache.sInstance.hasLifecycleMethods(klass);
        if (hasLifecycleMethods) {
            return REFLECTIVE_CALLBACK;
        }

        Class<?> superclass = klass.getSuperclass();
        List<Constructor<? extends GeneratedAdapter>> adapterConstructors = null;
        if (isLifecycleParent(superclass)) {
            if (getObserverConstructorType(superclass) == REFLECTIVE_CALLBACK) {
                return REFLECTIVE_CALLBACK;
            }
            adapterConstructors = new ArrayList<>(sClassToAdapters.get(superclass));
        }

        for (Class<?> intrface : klass.getInterfaces()) {
            if (!isLifecycleParent(intrface)) {
                continue;
            }
            if (getObserverConstructorType(intrface) == REFLECTIVE_CALLBACK) {
                return REFLECTIVE_CALLBACK;
            }
            if (adapterConstructors == null) {
                adapterConstructors = new ArrayList<>();
            }
            adapterConstructors.addAll(sClassToAdapters.get(intrface));
        }
        if (adapterConstructors != null) {
            sClassToAdapters.put(klass, adapterConstructors);
            return GENERATED_CALLBACK;
        }

        return REFLECTIVE_CALLBACK;
    }    


    /**
    获取 Observer 对应的 adapter 的构造方法
    */
    @SuppressWarnings("deprecation")
    @Nullable
    private static Constructor<? extends GeneratedAdapter> generatedConstructor(Class<?> klass) {
        try {
            Package aPackage = klass.getPackage();
            String name = klass.getCanonicalName();
            final String fullPackage = aPackage != null ? aPackage.getName() : "";
            // adapterName 是 Observer 的 adpter 的类名，如果 Observer 的名字为 A 那么通过 getAdapterName 方法可以知道 adaper 的名字就为 A_adapter
            final String adapterName = getAdapterName(fullPackage.isEmpty() ? name :
                    name.substring(fullPackage.length() + 1));
            // aClass 就是 adapter 的 class
            @SuppressWarnings("unchecked") final Class<? extends GeneratedAdapter> aClass =
                    (Class<? extends GeneratedAdapter>) Class.forName(
                            fullPackage.isEmpty() ? adapterName : fullPackage + "." + adapterName);
            // 获取构造方法                
            Constructor<? extends GeneratedAdapter> constructor =
                    aClass.getDeclaredConstructor(klass);
            if (!constructor.isAccessible()) {
                constructor.setAccessible(true);
            }
            return constructor;
        } catch (ClassNotFoundException e) {
            return null;
        } catch (NoSuchMethodException e) {
            // this should not happen
            throw new RuntimeException(e);
        }
    }    
    //通过 Observer 的名字获取 adapter 的类名
    public static String getAdapterName(String className) {
        return className.replace(".", "_") + "_LifecycleAdapter";
    }   
```