## 如何编写一个 Gradle 插件
### 项目结构
![Plugin 项目结构](https://raw.githubusercontent.com/ooftf/Material/master/img/blog/725b9022bc6364dc4cec8aba0416f07.png)

build.gradle 
###
```groovy
apply plugin: 'groovy'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    //gradle-api-6.7.1 对应 gradle/wrapper/gradle-wrapper.properties 文件内声明的版本 
    implementation gradleApi()  
}

```
### ooftf-spy.properties
```properties
implementation-class=com.ooftf.spy.plugin.ApiInspectPlugin
```
### ApiInspectPlugin
```groovy
package com.ooftf.spy.plugin
import org.gradle.api.Plugin
import org.gradle.api.Project

/**
 * Created by ooftf on 2021/3/2.
 */
class ApiInspectPlugin implements Plugin<Project> {

    @Override
    void apply(Project project) {
        // TODO SOMETING
    }
}
```
### Extension
#### ApiInspectExtension
```groovy
package com.ooftf.spy.plugin

import org.gradle.api.Action
import org.gradle.api.Project
import org.gradle.api.model.ObjectFactory

/**
 * Created by ooftf on 2021/3/2.
 */
class ApiInspectExtension {

    boolean enable = true
    boolean inspectSystemApi = false
    boolean interruptBuild = true
    ApiInspectExcludeExtension exclude
    ApiInspectExtension(Project project) {
        try {
            ObjectFactory objectFactory = project.getObjects()
            exclude = objectFactory.newInstance(ApiInspectExcludeExtension.class)
        } catch (Exception e) {
            exclude = ApiInspectExcludeExtension.class.newInstance()
        }
    }
    void exclude(Action<ApiInspectExcludeExtension> action) {
        action.execute(exclude)
    }
}

```
#### ApiInspectExcludeExtension
```groovy
package com.ooftf.spy.plugin
import com.google.common.base.Strings

/**
 * Created by ooftf on 2021/3/2.
 */
class ApiInspectExcludeExtension {
    Set<FilterItem> apis = new HashSet<>()
    void api(String name = "", String occurName = "", String method = "", String lineNumber = "") {
        if (Strings.isNullOrEmpty(name) && Strings.isNullOrEmpty(occurName) && Strings.isNullOrEmpty(method) && Strings.isNullOrEmpty(lineNumber))
            return
        FilterItem item = new FilterItem()
        item.name = name
        item.occurName = occurName
        item.method = method
        item.lines = lineNumber
        apis.add(item)
    }
}

```

#### 获取 Extension 数据
```groovy
class ApiInspectPlugin implements Plugin<Project> {

    @Override
    void apply(Project project) {
        project.extensions.create("spy", ApiInspectExtension.class, project)    
        ApiInspectExtension apiInspectExtension = project.extensions.findByType(ApiInspectExtension.class)
    }
}

```

## 使用插件 gradle.build
```groovy
apply plugin: 'ooftf-spy' // ooftf-spy 对应\resources\META-INF\gradle-plugins\ooftf-spy.properties 的文件名
// spy 对应  project.extensions.create("spy", ApiInspectExtension.class, project) 第一个参数"spy"
// spy 内部结构对应 ApiInspectExtension 的结构
spy {
    enable true //Whether api inspect is enabled.
    inspectSystemApi false //Whether to inspect the system api.
    interruptBuild false //Whether interrupt build when find error
    // 对应 ApiInspectExtension.exclude(Action<ApiInspectExcludeExtension> action)
    exclude {
        // 对应 ApiInspectExcludeExtension.api(String name = "", String occurName = "", String method = "", String lineNumber = "")
        api 'com.blankj.utilcode.', 'com.didichuxing.doraemonkit.'
        api 'com.blankj.utilcode.', 'com.didichuxing.doraemonkit.'
        api 'okhttp3.internal.platform.Platform', 'okhttp3.logging.HttpLoggingInterceptor'
        api 'kotlinx.atomicfu.InterceptorKt', 'kotlinx.atomicfu.AtomicFU'
    }
}
```

### 指定 plugin 名字的两种方式,选其一

#### 1. \resources\META-INF\gradle-plugins\ooftf-spy.properties
```properties
implementation-class=com.ooftf.spy.plugin.ApiInspectPlugin
```
#### 2. build.groovy （新，推荐）

```groovy
plugins {
    id 'java-gradle-plugin'
}
gradlePlugin {
    plugins {
        simplePlugin {
            id = 'ooftf-spy'
            implementationClass = 'com.ooftf.spy.plugin.ApiInspectPlugin'
        }
    }
}

```
