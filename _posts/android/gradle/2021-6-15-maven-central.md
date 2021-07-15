---
title: MavenCentral 配置
---

### 发布 snapshot 版本配置

```groovy
apply plugin: 'maven-publish'

afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                from components.release
                groupId = 'io.github.xxxxx'  //sonatype平台创建的groupId
                artifactId = project.name
                version = '1.0.0'  //版本,快照版本需要添加后缀 -SNAPSHOT
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
}
```

### 发布 release 版本配置

```groovy
apply plugin: 'maven-publish'
apply plugin: 'signing'
ext["signing.keyId"] = "*****"  //GPG指纹后8位
ext["signing.password"] = "*****"  //GPG密码
ext["signing.secretKeyRingFile"] = "/xxxxxx/xxxxx/xxxx.gpg" //GPG私钥文件
afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                from components.release
                groupId = 'io.github.xxxxx'  //sonatype平台创建的groupId
                artifactId = project.name
                version = '1.0.0'  //版本,快照版本需要添加后缀 -SNAPSHOT

                pom {
                    name = project.name// 推荐和 artifactId 相同
                    description = "noting to description"
                    url = "https://github.com/xxxxx"  //项目主页地址，可用 GitHub
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

### 终极版配置

```groovy
apply plugin: 'maven-publish'
apply plugin: 'signing'
task('generateSourcesJar', type: Jar) {
            from project.android.sourceSets.main.java.srcDirs
                classifier 'sources'
}
task('androidJavadocs', type: Javadoc) {
    source = project.android.sourceSets.main.java.srcDirs
    failOnError = false
    if (JavaVersion.current().isJava8Compatible()) {
        project.allprojects {
            project.tasks.withType(Javadoc) {
                enabled = false
                options.addStringOption('Xdoclint:none', '-quiet')
                options.setCharSet('UTF-8')
                options.setEncoding("UTF-8")
            }
        }
    }
}

task('javadocJar', type: Jar) {
    from project.tasks.androidJavadocs
    archiveClassifier = 'javadoc'
}

ext["signing.keyId"] = "*****"  //GPG指纹后8位
ext["signing.password"] = "*****"  //GPG密码
ext["signing.secretKeyRingFile"] = "/xxxxxx/xxxxx/xxxx.gpg" //GPG私钥文件
afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                from components.release
                groupId = 'io.github.xxxxx'  //sonatype平台创建的groupId
                artifactId = project.name
                version = '1.0.0'  //版本,快照版本需要添加后缀 -SNAPSHOT
                artifact javadocJar
                artifact generateSourcesJar
                pom {
                    name = project.name// 推荐和 artifactId 相同
                    description = "noting to description"
                    url = "https://github.com/xxxxx"  //项目主页地址，可用 GitHub
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