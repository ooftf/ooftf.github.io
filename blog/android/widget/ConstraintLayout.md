* layout_constrainedHeight  可以防止子View在wrap模式下超出父控件
    当一个控件设为wrap_content时，再添加约束尺寸是不起效果的。如需生效，需要设置如下属性为true:
    app:layout_constrainedWidth=”true|false”  
    app:layout_constrainedHeight=”true|false”
* layout_constraintVertical_bias
* layout_constraintHeight_default
    只有在 android:layout_height="0dp" 的时候 layout_constraintHeight_default 才起效
### 常见问题
    如果子控件设置Wrap，即使使用
    app:layout_constraintTop_toBottomOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"
    限制了边界，在控件内容过大的时候还是会超出边界

### 如果一个RecyclerView有以下几点要求该如何实现
* 在item比较少的情况下自身要采用wrap模式
* 要紧挨着上锚点  及 layout_constraintTop_toBottomOf的对象
* 在item比较多的情况不能超出父控件
* 同级控件还有个toolbar 要在toolbar下方

        方案
        使用
        app:layout_constraintTop_toBottomOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        android:layout_height="0dp"
        约束布局方式防止控件超出父控件，导致下面Item无法显示
        使用
        layout_constraintHeight_default
        让其在在item比较少的情况下要采用wrap模式
        使用
        layout_constraintVertical_bias="0"
        让其紧挨上面锚点

