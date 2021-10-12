NesedScroll 和 传统触摸事件分发处理有什么优势？
* 对滑动方向、fling、滑动距离、做了封装，处理起来更加方便。每个方法都有明确的意义
* 传统事件分发，在处理 “一个触摸循环” 两个控件分别处理一部分事件时，非常难处理。而 NestedScroll 比较好处理。
```java
   /**
     * 有嵌套滑动到来了，判断父控件是否接受嵌套滑动
     *
     * @param child            嵌套滑动对应的父类的子类(因为嵌套滑动对于的父控件不一定是一级就能找到的，可能挑了两级父控件的父控件，child的辈分>=target)
     * @param target           具体嵌套滑动的那个子类
     * @param nestedScrollAxes 支持嵌套滚动轴。水平方向，垂直方向，或者不指定
     * @return 父控件是否接受嵌套滑动， 只有接受了才会执行剩下的嵌套滑动方法
     */
    public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes) {}

    /**
     * 当onStartNestedScroll返回为true时，也就是父控件接受嵌套滑动时，该方法才会调用
     */
    public void onNestedScrollAccepted(View child, View target, int axes) {}

    /**
     * 在嵌套滑动的子控件未滑动之前，判断父控件是否优先与子控件处理(也就是父控件可以先消耗，然后给子控件消耗）
     *
     * @param target   具体嵌套滑动的那个子类
     * @param dx       水平方向嵌套滑动的子控件想要变化的距离 dx<0 向右滑动 dx>0 向左滑动
     * @param dy       垂直方向嵌套滑动的子控件想要变化的距离 dy<0 向下滑动 dy>0 向上滑动
     * @param consumed 这个参数要我们在实现这个函数的时候指定，回头告诉子控件当前父控件消耗的距离
     *                 consumed[0] 水平消耗的距离，consumed[1] 垂直消耗的距离 好让子控件做出相应的调整
     */
    public void onNestedPreScroll(View target, int dx, int dy, int[] consumed) {}

    /**
     * 嵌套滑动的子控件在滑动之后，判断父控件是否继续处理（也就是父消耗一定距离后，子再消耗，最后判断父消耗不）
     *
     * @param target       具体嵌套滑动的那个子类
     * @param dxConsumed   水平方向嵌套滑动的子控件滑动的距离(消耗的距离)
     * @param dyConsumed   垂直方向嵌套滑动的子控件滑动的距离(消耗的距离)
     * @param dxUnconsumed 水平方向嵌套滑动的子控件未滑动的距离(未消耗的距离)
     * @param dyUnconsumed 垂直方向嵌套滑动的子控件未滑动的距离(未消耗的距离)
     */
    public void onNestedScroll(View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) {}

    /**
     * 嵌套滑动结束
     */
    public void onStopNestedScroll(View child) {}

    /**
     * 当子控件产生fling滑动时，判断父控件是否处拦截fling，如果父控件处理了fling，那子控件就没有办法处理fling了。
     *
     * @param target    具体嵌套滑动的那个子类
     * @param velocityX 水平方向上的速度 velocityX > 0  向左滑动，反之向右滑动
     * @param velocityY 竖直方向上的速度 velocityY > 0  向上滑动，反之向下滑动
     * @return 父控件是否拦截该fling
     */
    public boolean onNestedPreFling(View target, float velocityX, float velocityY) {}


    /**
     * 当父控件不拦截该fling,那么子控件会将fling传入父控件
     *
     * @param target    具体嵌套滑动的那个子类
     * @param velocityX 水平方向上的速度 velocityX > 0  向左滑动，反之向右滑动
     * @param velocityY 竖直方向上的速度 velocityY > 0  向上滑动，反之向下滑动
     * @param consumed  子控件是否可以消耗该fling，也可以说是子控件是否消耗掉了该fling
     * @return 父控件是否消耗了该fling
     */
    public boolean onNestedFling(View target, float velocityX, float velocityY, boolean consumed) {}

    /**
     * 返回当前父控件嵌套滑动的方向，分为水平方向与，垂直方法，或者不变
     */
    public int getNestedScrollAxes() {}

```
不要使用 requestLayout 响应手指的滑动，会造成滑动数据抖动，不清楚为什么


使用 OverScroller 处理 fling 事件，具体用法如下
```java
    {
        OverScroller.fling(0,
                        getScrollY(),
                        (int)velocityX,
                        (int)velocityY,
                        0,
                        0,
                        0,
                        getCanScrollDistance());
                postInvalidate();
    }

// computeScroll  也可以用作监听界面的滚动事件
    @Override
    public void computeScroll() {
        //动画未完成
        if (scroller.computeScrollOffset()) {
            scrollTo(0, scroller.getCurrY());
            //触发draw()，必须有这一步
            postInvalidate();
        }
    }

```

## 三个基本不用重写的方法

* getNestedScrollAxes
* onStopNestedScroll
* onNestedScrollAccepted

这三个方法如果没有需要，可以不重写，因为 ViewGroup 已经编写了部分逻辑：主要是用于判断当前控件正在向哪个方向滚动，具体逻辑为：
onNestedScrollAccepted 记录滚动方向，onStopNestedScroll 重置为无方向，getNestedScrollAxes 获取当前滚动方向。