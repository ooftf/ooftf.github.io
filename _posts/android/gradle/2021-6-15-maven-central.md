---
title:MavenCentral 配置
---

```groovy
apply plugin: 'maven-publish'
apply plugin: 'signing'

afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                from components.release
                groupId = 'io.github.running-libo'  //sonatype平台创建的groupId
                artifactId = project.name
                version = '1.0.0'  //库版本名

                ext["signing.keyId"] = "*****"  //GPG指纹后8位
                ext["signing.password"] = "*****"  //GPG密码
                ext["signing.secretKeyRingFile"] = "/myfile/maven_upload_pub_sec/libo_0x55ECE4FE_SECRET.gpg" //GPG私钥文件

                pom {
                    name = "basemvvm"
                    description = "noting to description"
                    url = "https://github.com/running-libo"  //github主页地址
                    licenses {
                        license {
                            name = "The Apache License, Version 2.0"
                            url = "http://www.apache.org/licenses/LICENSE-2.0.txt"
                        }
                    }
                    developers {
                        developer {  //随意填开发者信息
                            id = "**"
                            name = "****"
                            email = "******"
                        }
                    }
                    scm {
                        connection = "scm:svn:http://github.com/xxxx"   //项目联系地址
                        developerConnection = "scm:svn:https://github.com/xxxxx"  //项目仓库地址
                        url = "http://github.com/xxx" //项目主页地址
                    }
                }
            }
        }

        repositories {
            maven {
                // 名字可随意，会显示在 task 中
                name = 'sonatypeRepository'
                //url = "https://s01.oss.sonatype.org/content/repositories/snapshots/" //快照上传地址
                // url = "https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/" 正式服务器
                url = "https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/"
                credentials {
                    //sonatype平台账号密码 密码有特殊字符可用 \ 转义
                    username = "***"
                    password = "********"
                }
            }
        }
    }
    signing {
        sign publishing.publications
    }
}
```