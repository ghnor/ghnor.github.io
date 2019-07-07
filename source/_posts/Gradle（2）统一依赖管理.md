---
moreLoc: 38
path: Android/Gradle（2）统一依赖管理.md
title: Gradle（2）统一依赖管理
date: 2017-12-01T15:25:01.000Z
updated: 2017-12-01T15:25:01.000Z
tags:
    - Android
categories:
    - Android
---

避免在依赖包出新版本时，需要对每个module中的build.gradle文件都进行修改（如appcompat-v7包），使用这种方式即只需一次修改。

## 在build.gradle中管理

首先第一种方式，直接在**根目录下的build.gradle**中组织我们的依赖：

```gradle
// 管理的依赖添加在这里
ext {
    supportVersion = '25.3.1'

    android = [ compileSdkVersion: 25 ,
                buildToolsVersion: "25.0.2",
                applicationId    : "com.gradle.test",
                minSdkVersion    : 15,
                targetSdkVersion : 25,
                versionCode      : 1,
                versionName      : "1.0" ]

    dependencies = [
            //android.support
            "appcompat-v7"      : "com.android.support:appcompat-v7:${supportVersion}",
            "design"            : "com.android.support:design:${supportVersion}",
            "recyclerview-v7"   : "com.android.support:recyclerview-v7:${supportVersion}",
    ]
}

// 原来就有的
buildscript {
    ...
}

// 原来就有的
allprojects {
    ...
}
```


<!--more-->

使用的时候，修改**各自module下的build.gradle（每个project起码有一个app module）**：

```gradle
android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion
    defaultConfig {
        applicationId rootProject.ext.android.applicationId
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        versionCode rootProject.ext.android.versionCode
        versionName rootProject.ext.android.versionName
    }

    ...
}

dependencies {
    ...

    compile rootProject.ext.dependencies["appcompat-v7"]
    // or: compile "com.android.support:appcompat-v7:${supportVersion}"
    
    ...
}
```

## 提取依赖到新的文件中管理

如果嫌弃都房子啊在build.gradle中太乱，就可以新建一个文件，把依赖都统一到新文件中去管理。

比如我们创建一个**新的config.gradle文件**，内容跟在**根目录下的build.gradle**添加的是一样的：

```gradle
ext {
    supportVersion = '25.3.1'

    android = [ compileSdkVersion: 25 ,
                buildToolsVersion: "25.0.2",
                applicationId    : "com.gradle.test",
                minSdkVersion    : 15,
                targetSdkVersion : 25,
                versionCode      : 1,
                versionName      : "1.0" ]

    dependencies = [
            //android.support
            "appcompat-v7"      : "com.android.support:appcompat-v7:${supportVersion}",
            "design"            : "com.android.support:design:${supportVersion}",
            "recyclerview-v7"   : "com.android.support:recyclerview-v7:${supportVersion}",
    ]
}
```

**在使用之前，需要先在根目录下的build.gradle中引用config.gradle：**

```gradle
apply from: "config.gradle" // 添加到此处

// 原来就有的
buildscript {
    ...
}

// 原来就有的
allprojects {
    ...
}
```

用法跟上面一样，不赘述。

## 在gradle.properties中管理

在gradle.properties中添加：

```properties
support_version = 25.3.1

compile_sdk_version = 25
build_tools_version = 25.0.2
min_sdk_version = 15
target_sdk_version = 25
```

修改app\build.gradle：

```gradle
...

buildToolsVersion build_tools_version

...

compile "com.android.support:appcompat-v7:${supportVersion}"

...
```

有一点要注意的是，比如说build.gradle中compileSdkVersion的数值本身应该是int类型的，但是如果从gradle.properties中读取的话，默认是string类型，所以build.gradle文件中原先值类型为int的参数需要在后面加上`as int`。

