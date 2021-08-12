---
published: true
title: ReentrantLock
---

## 自定义 Lock 的思路
定义一个 int 变量 state ，0: 表示锁没有被占用 ，大于0：表示锁已经被占用 

定义一个队列 用于储存等待中的线程

当 Lock 调用 Lock() 申请锁的时候，先判断当前 state 是否被占用（值为 0 ），如果已经被占用，加入到阻塞队列头节点，调用 LockSupport.park 阻塞当前线程；如果没有被占用，尝试使用 cas 将 state 加一   ，如果占用失败 再走锁被占用的流程

unLock():将 state 减一 ，判断减一后是否为 0 （即锁释放），如果为 0 ，从队列尾节点中取出头节点，执行 LockSupport.unpark




### 公平锁和非公平锁是如何实现的

```java

    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        // Android-removed: @ReservedStackAccess from OpenJDK 9, not available on Android.
        // @ReservedStackAccess
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }

    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        // Android-removed: @ReservedStackAccess from OpenJDK 9, not available on Android.
        // @ReservedStackAccess
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

总结：

公平锁和非公平锁的差异在于，

非公平锁调用 lock 的时候，不会判断队列是否为空，而是直接尝试用 cas 抢占锁。

而 公平锁 调用 lock 的时候，会先判断队列是否为空，如果为空才会用 cas 抢占锁，如果是队列不为空，会将当前线程添加到阻塞队列头部，阻塞当前线程

## AQS (AbstractQueuedSynchronizer)

### 底层实现

CAS + LockSupport.park

LockSupport.park 底层是调用的 Unsafe.park