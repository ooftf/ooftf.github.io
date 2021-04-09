# compose
[Google 文档](https://developer.android.google.cn/jetpack/compose/mental-model?hl=zh_cn)  
在应用或模块的 build.gradle 文件中添加所需工件的依赖项：
```groovy
android {
    buildFeatures {
        compose true
    }

    composeOptions {
        kotlinCompilerVersion "1.4.30"
        kotlinCompilerExtensionVersion "1.0.0-beta01"
    }
}

tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).configureEach {
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```
[Compose Google 使用文档](https://developer.android.google.cn/jetpack/compose?hl=zh_cn)

[组件重组策略](https://developer.android.google.cn/jetpack/compose/lifecycle?hl=zh_cn)
### Compose中添加AndroidView
```kotlin
@Composable
fun CustomView() {
    val selectedItem = remember { mutableStateOf(0) }

    // Adds view to Compose
    AndroidView(
        modifier = Modifier.fillMaxSize(), // Occupy the max size in the Compose UI tree
        viewBlock = { context ->
            // Creates custom view
            CustomView(context).apply {
                // Sets up listeners for View -> Compose communication
                myView.setOnClickListener {
                    selectedItem.value = 1
                }
            }
        },
        update = { view ->
            // View's been inflated or state read in this block has been updated
            // Add logic here if necessary

            // As selectedItem is read here, AndroidView will recompose
            // whenever the state changes
            // Example of Compose -> View communication
            view.coordinator.selectedItem = selectedItem.value
        }
    )
}

@Composable
fun ContentExample() {
    Column(Modifier.fillMaxSize()) {
        Text("Look at this CustomView!")
        CustomView()
    }
}
```
##  错误
var text by remember { mutableStateOf("123")}
报错：Type 'TypeVariable(T)' has no method 'getValue(Nothing?, KProperty<*>)' and thus it cannot serve as a delegate
添加 import androidx.compose.runtime.*

###  注意项
如果同一布局中存在多个 ComposeView 元素，每个元素必须具有唯一的 ID 才能使 savedInstanceState 发挥作用。