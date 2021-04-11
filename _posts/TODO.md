* ASM 修改class文件
* Transform API    获取打包过程中的class文件
* javapoet        一个用来动态生成Java文件的API
* annotationProcessor    注解处理器，能获取到写有注解的类

#  Transform API + ASM  可以做AOP
#  annotationProcessor + javapoet  可以根据注解生成一些辅助类

* 引用了两个库A和B，A调用了B一个方法，后来我升级了B库这个方法没有了，运行就会报错，这时候时可以打包成功的，怎么在打包前发现这个问题呢？
