## SharedPreferences 代码分析
#### ContextImpl.getSharedPreferences
ContextImpl.getSharedPreferences(String name, int mode)
```java
@Override
public ContextImpl.SharedPreferences getSharedPreferences(String name, int mode) {
    if (mPackageInfo.getApplicationInfo().targetSdkVersion <
            Build.VERSION_CODES.KITKAT) {
        if (name == null) {
            name = "null";
        }
    }
    File file;
    synchronized (ContextImpl.class) {
        if (mSharedPrefsPaths == null) {
            mSharedPrefsPaths = new ArrayMap<>();
        }
        file = mSharedPrefsPaths.get(name);
        if (file == null) {
            file = getSharedPreferencesPath(name);
            mSharedPrefsPaths.put(name, file);
        }

    }

    // 上面代码就是通过 name 获取到对应的 file
    return getSharedPreferences(file, mode);
}
```
ContextImpl.getSharedPreferences(File file, int mode)
```java
@Override
public SharedPreferences getSharedPreferences(File file, int mode) {
    SharedPreferencesImpl sp;
    synchronized (ContextImpl.class) {
        final ArrayMap<File, SharedPreferencesImpl> cache =getSharedPreferencesCacheLocked(); 
        // 分析getSharedPreferencesCacheLocked 可知所有 Context 获取到的是同一个 cache 对象
        // File 对象如果路径相同那么File.equals 和 File.hashCode 也相同，所以可知在所有 Context 中 cache.get(file);获取到的 SharedPreferences 都是同一个对象
        sp = cache.get(file);
        if (sp == null) {
            checkMode(mode);
            sp = new SharedPreferencesImpl(file, mode);
            cache.put(file, sp);
            return sp;
        }
    }
    // 如果是多进程模式，每次获取 SharedPreferences 都会重新从磁盘中获取数据
    if ((mode & Context.MODE_MULTI_PROCESS) != 0 ||
        getApplicationInfo().targetSdkVersion < android.os.BuildVERSION_CODES.HONEYCOMB) {
        sp.startReloadIfChangedUnexpectedly();
    }
    return sp;
}
```
ContextImpl.getSharedPreferencesCacheLocked
```java
//ContextImpl.sSharedPrefsCache 是一个静态变量,因此所有Context 都是用的同一个 sSharedPrefsCache对象

private static ArrayMap<String, ArrayMap<File, SharedPreferencesImpl>> sSharedPrefsCache;

@GuardedBy("ContextImpl.class")
private ArrayMap<File, SharedPreferencesImpl>getSharedPreferencesCacheLocked() {
    
    if (sSharedPrefsCache == null) {
        sSharedPrefsCache = new ArrayMap<>();
    }
    final String packageName = getPackageName();
    // 因为 sSharedPrefsCache 在所有 Context 中都是同一个对象，所以 packagePrefs 对象也是同一个
    ArrayMap<File, SharedPreferencesImpl> packagePrefs =sSharedPrefsCache.get(packageName);
    if (packagePrefs == null) {
        packagePrefs = new ArrayMap<>();
        sSharedPrefsCache.put(packageName, packagePrefs);
    }
    return packagePrefs;
}
```

结论： 从上面分析可知 所有 Context.getSharedPreferences 如果 name 相同那么他们获取的是同一个对象

#### SharedPreferencesImpl

```java
private Map<String, Object> mMap; //这个对象是存放 SharedPreferences 的 Key Value 值 
@UnsupportedAppUsage
SharedPreferencesImpl(File file, int mode) {
    mFile = file;
    mBackupFile = makeBackupFile(file);
    mMode = mode;
    mLoaded = false;
    mMap = null;
    mThrowable = null;
    startLoadFromDisk();// 从硬盘中读取 Key Value 并存放到 mMap 中
}

private void startLoadFromDisk() {
    synchronized (mLock) {
        mLoaded = false;
    }
    new Thread("SharedPreferencesImpl-load") { // 这是虽然开启了线程但是在诸如 getFloat 等方法中会通过 wait 的形式等待 加载完毕
        public void run() {
            loadFromDisk();// 在子线程中从硬盘中读取 Key Value 并存放到 mMap 中
        }
    }.start();
}


public float getFloat(String key, float defValue) {
    synchronized (mLock) {
        awaitLoadedLocked(); // 等待 loadFromDisk()完毕
        Float v = (Float)mMap.get(key);// 从内存中获取对应的值
        return v != null ? v : defValue;
    }
}
```
总结：从上面代码中得知 SharedPreferencesImpl 对象在构造方法中从硬盘获取到Map值，之后只要不主动调用Context.reloadSharedPreferences() 就不会再从硬盘中获取



 



#### EditorImpl 分析存入的过程

分析代码需要先看懂 [CountDownLatch](https://www.jianshu.com/p/e233bb37d2e6) 的作用

```java
//用来暂存通过 Editor 修改的内容
private final Map<String, Object> mModified = new HashMap<>();
// putString 只是将修改的内容暂存到  mModified 并没有将修改保存到 SharedPreferencesImpl.mMap 中，更没有保存到硬盘中
public Editor putString(String key, @Nullable String value) {
    synchronized (mEditorLock) {
        mModified.put(key, value);
        return this;
    }
}

@Override
public void apply() {
    final long startTime = System.currentTimeMillis()
    // 将修改提交到内存中
    final MemoryCommitResult mcr = commitToMemory();
    final Runnable awaitCommit = new Runnable() {
            @Override
            public void run() {
                try {
                    mcr.writtenToDiskLatch.await();//writtenToDiskLatch 是 CountDownLatch 类，当写入磁盘操作执行完成将 writtenToDiskLatch 置为0 ，才会继续向下执行。总的来说，就是等待写入磁盘操作执行完成
                } catch (InterruptedException ignored) {
                
            }
        }
    QueuedWork.addFinisher(awaitCommit)
    Runnable postWriteRunnable = new Runnable() {
            @Override
            public void run() {
                awaitCommit.run();
                QueuedWork.removeFinisher(awaitCommit);
            }
        }
    // // 将修改提交到硬盘中，因为第二个参数不为 null 所以这是个异步任务
    SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable)
    notifyListeners(mcr);
}

@Override
public boolean commit() {
    // 将修改提交到内存中
    MemoryCommitResult mcr = commitToMemory()
    // 将修改提交到硬盘中，因为第二个参数为 null 所以这是个同步任务
    SharedPreferencesImpl.this.enqueueDiskWrite(
        mcr, null);
    try {
        mcr.writtenToDiskLatch.await();// 等待写入磁盘操作执行完成
    } catch (InterruptedException e) {
        return false;
    } 
    notifyListeners(mcr);
    return mcr.writeToDiskResult;
}
//
private void SharedPreferencesImpl.enqueueDiskWrite(final MemoryCommitResult mcr,
                              final Runnable postWriteRunnable) {
    // 如果 postWriteRunnable 为 null 代表是一个同步任务                             
    final boolean isFromSyncCommit = (postWriteRunnable == null);
    final Runnable writeToDiskRunnable = new Runnable() {
            @Override
            public void run() {
                synchronized (mWritingToDiskLock) {
                    writeToFile(mcr, isFromSyncCommit);
                }
                synchronized (mLock) {
                    mDiskWritesInFlight--;
                }
                if (postWriteRunnable != null) {
                    postWriteRunnable.run();
                }
            }
        };
    // 如果是同步任务则直接执行，commit
    if (isFromSyncCommit) {
        boolean wasEmpty = false;
        synchronized (mLock) {
            wasEmpty = mDiskWritesInFlight == 1;
        }
        if (wasEmpty) {
            writeToDiskRunnable.run();
             // 同步任务执行完成直接 return
            return;
        }
    }
    // 异步任务执行方法，apply
    QueuedWork.queue(writeToDiskRunnable, !isFromSyncCommit);
}

public static void queue(Runnable work, boolean shouldDelay) {
    Handler handler = getHandler();

    synchronized (sLock) {
        sWork.add(work); //添加到 sWork 中  work 会在 Activity 执行 onStop onPause 的时候执行，或者 接收到  MSG_RUN 消息时执行

        if (shouldDelay && sCanDelay) {
            // DELAY 默认值为 100 ，会在 100 毫秒后执行 sWork 内的 work
            handler.sendEmptyMessageDelayed(QueuedWorkHandler.MSG_RUN, DELAY);
        } else {
            handler.sendEmptyMessage(QueuedWorkHandler.MSG_RUN);
        }
    }
}
```

总结：commit 方法是会直接将修改提交到内存和硬盘中的，apply 方法是先将修改提交到内存中，而提交到硬盘中的操作会在 Activity 执行 onStop onPause  或者 通过Hanlder 100毫秒后执行
#### 分析为什么  SharedPreferences 不能跨进程
由于同一个 name 的 SharedPreferences 只存在一个对象，并且 SharedPreferences 读取硬盘的操作，在正常流程中只会在执行构造方法的地方执行一次

子进程修改 SharedPreferences 内存中的值，因为进程间内存隔离的原因是不会反映到主进程中的，
修改硬盘中的值，如果主进程对应的 SharedPreferences 还没有创建，这次修改是可以反应到主进程中的，但是仅限这次修改；如果主进程 SharedPreferences 已经构造完成，就不会从硬盘中获取对应的数据，子进程修改的值也就不会反应到子进程中。

即使将模式改为 MODE_MULTI_PROCESS ，每次调用 Context.getSharedPreferences 都会检查磁盘文件是否修改，如果已经修改重新从磁盘读取数据，这个方式不仅会增加获取 getSharedPreferences 的时间消耗，也会因为多进程并发修改文件而产生安全问题。推荐使用 ContentProvider 、跨进程通讯封装、MMKV

所以总的来说 SharedPreferences 是不能跨进程的
