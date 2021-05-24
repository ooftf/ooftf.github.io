# [ViewBinding](https://developer.android.com/topic/libraries/view-binding)
### gradle 配置
视图绑定功能可按模块启用。要在某个模块中启用视图绑定，请将 viewBinding 元素添加到其 build.gradle 文件中，如下例所示：
```groovy
android {
    ...
    buildFeatures {
        viewBinding true
    }
}
```

### 使用说明
为某个模块启用视图绑定功能后，系统会为该模块中包含的每个 XML 布局文件生成一个绑定类。每个绑定类均包含对根视图以及具有 ID 的所有视图的引用。系统会通过以下方式生成绑定类的名称：将 XML 文件的名称转换为驼峰式大小写，并在末尾添加“Binding”一词。

```xml
<!--result_profile.xml-->
<LinearLayout ... >
    <TextView android:id="@+id/name" />
    <ImageView android:cropToPadding="true" />
    <Button android:id="@+id/button"
        android:background="@drawable/rounded_button" />
</LinearLayout>
    
```
#### 在 Activity 中使用
```kotlin
    private lateinit var binding: ResultProfileBinding

    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate(savedInstanceState)
        binding = ResultProfileBinding.inflate(layoutInflater)
        val view = binding.root
        setContentView(view)
    }
    
```

#### 在 Fragment 中使用视图绑定
```kotlin
    private var _binding: ResultProfileBinding? = null
    // This property is only valid between onCreateView and
    // onDestroyView.
    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = ResultProfileBinding.inflate(inflater, container, false)
        val view = binding.root
        return view
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
    
```

#### 其他
如果您希望在生成绑定类时忽略某个布局文件，请将 tools:viewBindingIgnore="true" 属性添加到相应布局文件的根视图中：
```xml
<LinearLayout
            ...
            tools:viewBindingIgnore="true" >
        ...
    </LinearLayout>
    
```

## 与 DataBinding 相比较
ViewBinding 和 DataBinding 均会生成可用于直接引用视图的绑定类。但是，ViewBinding 旨在处理更简单的用例，与 DataBinding 相比，具有以下优势：

* 更快的编译速度：ViewBinding不需要处理注释，因此编译时间更短。
* 易于使用：ViewBinding 不需要特别标记的 XML 布局文件，因此在应用中采用速度更快。在模块中启用 ViewBinding 后，它会自动应用于该模块的所有布局。

反过来，与DataBinding相比，ViewBinding 也具有以下限制：
* ViewBinding 不支持布局变量或布局表达式，因此不能用于直接在 XML 布局文件中声明动态界面内容。
* ViewBinding 不支持双向 DataBinding。

考虑到这些因素，在某些情况下，最好在项目中同时使用 ViewBinding 和 DataBinding。您可以在需要高级功能的布局中使用 DataBinding，而在不需要高级功能的布局中使用 ViewBinding。

## 生成的 Binding 的位置
\build\generated\data_binding_base_class_source_out\debug\out\{applicationId}\databinding\{name}Binding.java