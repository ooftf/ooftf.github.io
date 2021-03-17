# View
## View的移动
* 不改变布局参数(不会触发layout)
  1. scrollTo
  2. 传统动画和属性动画（translationX translationY）
* 改变布局参数(改变LayoutParams)
## 平滑滑动动画
* Scroller
* ObjectAnimator
# View的事件分发

* dispatchTouchEvent
* onInterceptTouchEvent
* setOnTouchEventListener
* onTouchEvent

```kotlin
//伪代码
    fun dispatchTouchEvent(ev:MotionEvent):Boolean{
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            mFirstTouchTarget = null
        }
        if(MotionEvent.ACTION_DOWN|| mFirstTouchTarget != null){
            intercepted = onInterceptTouchEvent(ev);
        }
        if(!intercepted){
            if(actionMasked == MotionEvent.ACTION_DOWN){
                if(child.dispatchTouchEvent(ev)){
                    addTouchTarget(child)//mFirstTouchTarget赋值为TouchTarget的头节点
                }
            }
        }
        if(mFirstTouchTarget == null){
            // 执行View的touch系列事件
            return onTouchEvent(ev)
        }else{
            //有个避免ACTION_DOWN二次调用的逻辑没有写
            var handler
            while(mFirstTouchTarget.hasNext){
                if(child.dispatchTouchEvent(ev)){
                    handler = true
                }
            }
            return handler
            //执行TouchTarget链表内View的事件
        }
    }
```
```

```
### 从TouchTarget分析特性  mFirstTouchTarget是TouchTarget列表的头
* 如果 onInterceptTouchEvent如果返回true -> mFirstTouchTarget = null   ->下个event不会进入onInterceptTouchEvent
* onInterceptTouchEvent返回false -> mFirstTouchTarget被赋值 -> event进入onInterceptTouchEvent
* 如果mFirstTouchTarget 为null 则会执行父View的touch事件如果mFirstTouchTarget不为null则会执行TouchTarget之内的子View的touch事件
* 生成mFirstTouchTarget链表的动作只有在  ACTION_DOWN  ACTION_POINTER_DOWN ACTION_HOVER_MOVE 事件产生
* 如果在 ACTION_DOWN 事件时子View的dispatchTouchEvent 返回为false就会导致 该View没有加入到 mFirstTouchTarget链表中，就会导致后续接收不到事件
------
总结：mFirstTouchTarget是否为null是由 ACTION_DOWN 时子View dispatchTouchEvent的返回值决定的
问题：mFirstTouchTarget为什么是个链表？一个事件可以由多个Child处理?
### 特性
* 如果View设置了onTouchListener，onTouchListener是优先于onTouchEvent，如果onTouchListener返回true那么就不会调用onTouchEvent方法，如果返回false就会执行onTouchEvent
* onClickListener等事件都是在onTouchEvent种执行的
* 如果View不消耗ACTION_DOWN(dispatchTouchEvent返回false)那么同一时间序列中的其他事件事件都不会交给他来处理，并且事件将重新交由他的父元素去处理。即父元素的onTouchEvent会被调用
* 如果View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onToucheEvent并不会被调用，并且当前View可以持续收到后续的事件
## 滑动冲突
###  常见的滑动冲突
* 外部滑动方向和内部滑动方向不一致
* 外部滑动方向和内部滑动方向一直
* 上面两种情况叠加嵌套
