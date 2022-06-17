---
title: Android使用Maven Publish插件打包发布到私有Maven仓库
categories: -技术
tags: 
- Maven
- Gradle 
- Android
date: 2022-01-06
---

# Android 使用 Maven Publish 插件打包发布到私有Maven仓库

## ~~Android Gradle 3.6.0之前~~

```groovy
apply plugin: 'maven'
uploadArchives {
    repositories {
        mavenDeployer {
            pom.artifactId = '项目信息'
            pom.version = '版本信息'
            repository(url: '私服仓库地址') {
                authentication(userName: '账号', password: '密码')
            }
            snapshotRepository(url: '私服快照地址') {
               authentication(userName: '账号', password: '密码')
            }
        }
    }
}
```



##  Android Gradle 3.6.0 之后

Android Gradle 插件 3.6.0之后，[Google推荐使用Maven Publish插件](https://developer.android.com/studio/build/maven-publish-plugin)，但官方给出的示例过于简单，并没有私有库相关发布方法


## 发布

### 1.添加Maven Publish Plugin

在将要发布模块的build.gradle文件中添加 maven-publish 插件

```gradle
plugins {
    id 'com.android.library'
    id 'kotlin-android'
    id 'maven-publish'
}
```

### 2.定义publishing Task发布ARR

大部分场景下Android会选择发布ARR提供可复用代码	

以下代码示例为 AAR 库的发布，示例中包含了debug和release两种维度的发布，当然有时候我们仅仅发布release

```groovy
afterEvaluate {
    publishing {
        publications {
            debug(MavenPublication) {
            	//相应变体
                from components.debug
                // 定义发布属性，一般为模块包名
                groupId =  'com.example.upgrade'
                //一般填写项目信息比如utlis,upgrade等功能性简洁描述
                artifactId = 'upgrade-debug'
                //版本号
                version = '1.0'
            }
            release(MavenPublication) {
                from components.release
                groupId =  'com.example.upgrade'
                artifactId = 'upgrade'
                version = '1.0'
            }
        }
        //设置存储库信息
        repositories {
            maven {
                url = MAVEN_URL //私有的Maven库地址
                //allowInsecureProtocol = true//允许非https链接
                credentials {
                    username = MAVEN_PRIVATE_USERNAME//Maven分配的用户名
                    password = MAVEN_PRIVATE_PASSWORD//用户名对应的密码
                }
            }
        }
    }
}
```

> [Gradle Maven Publish Plugin详细说明](https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven:usage)


## 使用

### 1.设置私有仓库地址

在项目根目录中settings.gradle文件中设置私有Maven地址

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven {
            url MAVEN_PRIVATE//私有maven服务
            //允许非https链接
            //allowInsecureProtocol = true
            //当maven开启严格验证的时候需要填写用户名和密码
            credentials {
                username MAVEN_PRIVATE_USERNAME
                password MAVEN_PRIVATE_PASSWORD
            }
        }
    }
}
rootProject.name = "MavenTest"
include ':app'
```

### 2.引用已上传的ARR库

在app或使用的模块目录build.gradle文件中添加库

```groovy
dependencies {
    implementation 'com.example.upgrade:upgrade:1.0'
}
```

## 参考
[Gradle MavenPulish Plugin](https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven:usage)

[Google示例](https://developer.android.com/studio/build/maven-publish-plugin)