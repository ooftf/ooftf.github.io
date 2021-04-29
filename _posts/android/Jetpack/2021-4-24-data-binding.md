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
  即使 App Module 没有使用 dataBinding 也要配置 dataBinding
  ```groovy
  dataBinding {
      nabled true
  }
  ```

