## 强制使用指定版本
configurations.all {

    resolutionStrategy {
        force 'com.android.support:appcompat-v7:26.1.0'
        force "com.android.support:support-v4:${supportVersion}"
        force "com.android.support:appcompat-v7:${supportVersion}"
        force "com.android.support:design:${supportVersion}"
        force "com.android.support:recyclerview-v7:${supportVersion}"
        force "com.android.support:cardview-v7:${supportVersion}"
        force "com.android.support:gridlayout-v7:${supportVersion}"
        force "com.android.support:support-annotations:${supportVersion}"
    }
}
## 排除指定引用
dependencies {
    implementation('some-library') {
        exclude group: 'com.example.imgtools', module: 'native'
    }
}
## 在所有库中排除指定引用
android.testVariants.all { variant ->
    variant.getCompileConfiguration().exclude group: 'com.jakewharton.threetenabp', module: 'threetenabp'
    variant.getRuntimeConfiguration().exclude group: 'com.jakewharton.threetenabp', module: 'threetenabp'
}
exclude 在pom文件中不会起作用，只是在本项目中起作用，如果打包成maven依赖，exclude不起作用

transitive = false

publishDebugPublicationToMavenLocal,publishToMavenLocal即不用debugImplementation也不用releaseImplementation

## 查看更多错误信息
    gradlew build --info --stacktrace --debug --scan


##
 kotlinOptions {
        jvmTarget = '1.8'
    }