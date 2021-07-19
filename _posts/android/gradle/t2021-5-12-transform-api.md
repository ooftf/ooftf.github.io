---
published: false
title: Transform API
---

## Transform API 是什么
Gradle Transform Api 只有在 Gradle Version 1.5+ 的时候才开放
[Transform](https://developer.android.com/reference/tools/gradle-api/7.1/com/android/build/api/transform/Transform)

## 怎么使用
#### 引入依赖
```groovy
// 为什么用 complieOnly ：因为如果采用 implementation 当使用者的 gradle 版本低于此版本就会导致用户 gradle 版本升级，这并不是我们想看到的

compileOnly 'com.android.tools.build:gradle:4.2.0'

```
#### 编写 Tranform
```groovy
package com.ooftf.spy.plugin
import com.google.common.collect.ImmutableSet
import com.android.build.api.transform.*

/**
 * Created by ooftf on 2021/3/2.
 */
class MyTransform extends Transform {

    // 定义一个名字
    @Override
    String getName() {
        return null
    }
    // 
    // 获取数据类型 
    // CLASSES 编译后的Java代码
    // RESOURCES Java 源码
    @Override 
    Set<QualifiedContent.ContentType> getInputTypes() {
        return ImmutableSet.of(QualifiedContent.DefaultContentType.CLASSES)
    }
    // 输出数据类型，默认和 getInputTypes 相同
    @Override
    Set<QualifiedContent.ContentType> getOutputTypes() {
        return super.getOutputTypes()
    }

    // 需要修改的部分，数据的来源范围，类型和 getReferencedScopes 相同
    @Override
    Set<? super QualifiedContent.Scope> getScopes() {
        return ImmutableSet.of()
    }

    // 仅仅是获取不作修改的部分，数据的来源范围
        /** Only the project (module) content */
        // PROJECT(0x01),
        /** Only the sub-projects (other modules) */
        // SUB_PROJECTS(0x04),
        /** Only the external libraries */
        // EXTERNAL_LIBRARIES(0x10),
        /** Code that is being tested by the current variant, including dependencies */
        // TESTED_CODE(0x20),
        /** Local or remote dependencies that are provided-only */
        // PROVIDED_ONLY(0x40),    
    @Override
    Set<? super QualifiedContent.Scope> getReferencedScopes() {
        return TransformManager.SCOPE_FULL_PROJECT
    }
    // 是否支持增量更新
    @Override
    boolean isIncremental() {
        return true
    }
    // 数据转换
    @Override
    void transform(TransformInvocation transformInvocation) throws TransformException, InterruptedException, IOException {
        
    }
}
```
#### 注册 Tranform 
```groovy
import com.android.build.gradle.AppExtension

class MyPlugin implements Plugin<Project> {

    @Override
    void apply(Project project) {
        def android = project.extensions.getByType(AppExtension.class)
        android.registerTransform(new MyTransform())
    }
}
```