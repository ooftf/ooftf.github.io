## 作用：

* CALLBACK_INPUT
* CALLBACK_ANIMATION
* CALLBACK_INSETS_ANIMATION
* CALLBACK_TRAVERSAL
* CALLBACK_COMMIT

根据是否使用垂直同步，分两种方式统一控制上面五种回调的执行时机

1. 根据垂直同步信号
2. 使用间隔固定时长？

## 使用
```kotlin

Choreographer.getInstance().postFrameCallback {

}

// 这是一个 Test Api 不对外开放
Choreographer.getInstance().postCallback(int callbackType, Runnable action, Object token)
```

#### 分析 getInstance 方法
```java
public static Choreographer Choreographer.getInstance() {
    return sThreadInstance.get();
}

private static final ThreadLocal<Choreographer> sThreadInstance =
        new ThreadLocal<Choreographer>() {
    @Override
    protected Choreographer initialValue() {
        Looper looper = Looper.myLooper();
        if (looper == null) {
            throw new IllegalStateException("The current thread must have a looper!");
        }
        Choreographer choreographer = new Choreographer(looper, VSYNC_SOURCE_APP);
        if (looper == Looper.getMainLooper()) {
            mMainInstance = choreographer;
        }
        return choreographer;
    }
};
```

总结：由代码可知，Choreographer.getInstance() 获取到的对象是保存在 ThreadLocal 中的，也就是不同的线程获取到的 Choreographer 实例不同。不同线程的 Choreographer 实例对象，使用当前线程的 Looper。

#### 分析 Choreographer.postFrameCallback 方法
```java
    private void postCallbackDelayedInternal(int callbackType,
            Object action, Object token, long delayMillis) {
        synchronized (mLock) {
            final long now = SystemClock.uptimeMillis();
            final long dueTime = now + delayMillis;

            //mCallbackQueues 是个数组 包含了下面五种类型
            //1. CALLBACK_INPUT 2. CALLBACK_ANIMATION 3. CALLBACK_INSETS_ANIMATION 4. CALLBACK_TRAVERSAL 5. CALLBACK_COMMIT

            mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);
           
            if (dueTime <= now) {
                //如果立即执行或者已经过了执行时间
                scheduleFrameLocked(now);
            } else {
                 // 如果还未到执行时间，利用 handler 延迟执行 scheduleFrameLocked
                Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
                msg.arg1 = callbackType;
                // 异步消息
                msg.setAsynchronous(true);
                //mHandler 是 FrameHandler
                mHandler.sendMessageAtTime(msg, dueTime);
            }
        }
    }
    private final class FrameHandler extends Handler {
        public FrameHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_DO_FRAME:
                    doFrame(System.nanoTime(), 0);
                    break;
                case MSG_DO_SCHEDULE_VSYNC:
                    doScheduleVsync();
                    break;
                case MSG_DO_SCHEDULE_CALLBACK:
                    doScheduleCallback(msg.arg1);
                    break;
            }
        }
    }
```
```java
    private void scheduleFrameLocked(long now) {
        if (!mFrameScheduled) {
            mFrameScheduled = true;
            if (USE_VSYNC) {// 如果使用了垂直同步
        
                if (isRunningOnLooperThreadLocked()) {
                     // 如果执行线程的 Looper 和 Choreographer 的 looper 是同一个looper
                    scheduleVsyncLocked();
                } else {
                    // 先发送到 Choreographer.looper 中，然后执行 scheduleVsyncLocked()
                    Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_VSYNC);
                    msg.setAsynchronous(true);
                    mHandler.sendMessageAtFrontOfQueue(msg);
                }
            } else {
                // 计算下一帧的时间
                final long nextFrameTime = Math.max(
                        mLastFrameTimeNanos / TimeUtils.NANOS_PER_MS + sFrameDelay, now);
                // 发送异步消息，执行 doFrame() 方法
                Message msg = mHandler.obtainMessage(MSG_DO_FRAME);
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, nextFrameTime);
            }
        }
    }
```
#### scheduleVsy
```java
    private void scheduleVsyncLocked() {
        mDisplayEventReceiver.scheduleVsync();
    }
    public void DisplayEventReceiver.scheduleVsync() {
        if (mReceiverPtr == 0) {
           
        } else {
            nativeScheduleVsync(mReceiverPtr);
        }
    }
    // 很明显这是一个 native 方法
    private static native void nativeScheduleVsync(long receiverPtr);   

    // 接受垂直同步消息
    @Override
    public void FrameDisplayEventReceiver.onVsync(long timestampNanos,long physicalDisplayId, int frame) {
        long now = System.nanoTime();
        if (timestampNanos > now) {
            timestampNanos = now;
        }
        if (mHavePendingVsync) {
        
        } else {
            mHavePendingVsync = true;
        }
        mTimestampNanos = timestampNanos;
        mFrame = frame;
        Message msg = Message.obtain(mHandler, this);
        msg.setAsynchronous(true);
        mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtilsNANOS_PER_MS);
    }
        @Override
        public void run() {
            mHavePendingVsync = false;
            doFrame(mTimestampNanos, mFrame);
        }    
```
总结：scheduleVsyncLocked 发送接受垂直同步信号，当接收到垂直同步信号会调用 FrameDisplayEventReceiver.onVsync，发送异步消息，回调到 run 方法，执行 doFrame 方法

``` java
void doFrame(long frameTimeNanos, int frame) {
    final long startNanos;
    synchronized (mLock) {
        if (!mFrameScheduled) {
            return; 
        }
        long intendedFrameTimeNanos = frameTimeNanos;
        startNanos = System.nanoTime();
        final long jitterNanos = startNanos - frameTimeNanos;
        if (jitterNanos >= mFrameIntervalNanos) {
            final long skippedFrames = jitterNanos /mFrameIntervalNanos;
            if (skippedFrames >= SKIPPED_FRAME_WARNING_LIMIT) {
                
            }
            final long lastFrameOffset = jitterNanos %mFrameIntervalNanos;

            frameTimeNanos = startNanos - lastFrameOffset;
        }
        if (frameTimeNanos < mLastFrameTimeNanos) {
            scheduleVsyncLocked();
            return;
        }
        if (mFPSDivisor > 1) {
            long timeSinceVsync = frameTimeNanos - mLastFrameTimeNanos;
            if (timeSinceVsync < (mFrameIntervalNanos * mFPSDivisor) & timeSinceVsync > 0) {
                scheduleVsyncLocked();
                return;
            }
        }
        mFrameInfo.setVsync(intendedFrameTimeNanos, frameTimeNanos);
        mFrameScheduled = false;
        mLastFrameTimeNanos = frameTimeNanos;
    }
    try {
        // 分别执行 5 种回调
        AnimationUtils.lockAnimationClock(frameTimeNanos / TimeUtilsNANOS_PER_MS);
        mFrameInfo.markInputHandlingStart();
        doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);
        mFrameInfo.markAnimationsStart();
        doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);
        doCallbacks(Choreographer.CALLBACK_INSETS_ANIMATION,frameTimeNanos);
        mFrameInfo.markPerformTraversalsStart();
        doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);
        doCallbacks(Choreographer.CALLBACK_COMMIT, frameTimeNanos);
    } finally {
        AnimationUtils.unlockAnimationClock();
    }
}
```