---
title: Gradle Build 构建配置
date: 2021-09-26 20:02
tags:
  - Gradle
categories: [Android,Gradle]
toc: true
updated: 2021-09-26 20:02
references:
  - title: 配置 build  | Android 开发者  | Android Developers
    url: https://developer.android.com/studio/build
  - title: 史上最全Android build.gradle配置详解，你懂的！ - 掘金 (juejin.cn)
    url: https://juejin.cn/post/6844903933584883720
---

Android 构建系统会编译应用资源和源代码，然后将它们打包到 APK 或 Android App Bundle 中，供测试、部署、签名和分发。

Android Studio 会使用高级构建工具包 [Gradle](http://www.gradle.org/) 自动执行和管理构建流程，同时也允许定义灵活的自定义 build 配置。

<!-- more -->

## 构建流程

典型 Android 应用模块的构建流程按照以下常规步骤执行：

<center>
<img src="https://developer.android.com/images/tools/studio/build-process_2x.png" alt=" 典型 Android 应用模块的构建流程 " style="zoom:50%;" />

</center>

1. 编译器将您的源代码转换成 DEX 文件（Dalvik 可执行文件，其中包括在 Android 设备上运行的字节码），并将其他所有内容转换成编译后的资源。
2. 打包器将 DEX 文件和编译后的资源组合成 APK 或 AAB（具体取决于所选的 build 目标）。 必须先为 APK 或 AAB 签名，然后才能将应用安装到 Android 设备或分发到 Google Play 等商店。
3. 打包器使用调试或发布密钥库为 APK 或 AAB 签名：
	- 如果您构建的是调试版应用（即专门用来测试和分析的应用），则打包器会使用调试密钥库为应用签名。Android Studio 会自动使用调试密钥库配置新项目。
	- 如果您构建的是打算对外发布的发布版应用，则打包器会使用发布密钥库（您需要进行配置）为应用签名。如需创建发布密钥库，请参阅 [在 Android Studio 中为应用签名](https://developer.android.com/studio/publish/app-signing#studio)。
4. 在生成最终 APK 之前，打包器会使用 [zipalign](https://developer.android.com/studio/command-line/zipalign) 工具对应用进行优化，以减少其在设备上运行时所占用的内存。

构建流程结束时，您将获得应用的调试版或发布版 APK/AAB，以用于部署、测试或向外部用户发布。

## 配置文件

### Gradle 设置文件

`settings.gradle` 文件位于项目的根目录下，用于指示 Gradle 在构建应用时应将哪些**模块**包含在内。

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "hhuer"
include(":app")
```

### 项目 Build 文件

顶层 `build.gradle` 文件位于项目的根目录下，用于定义适用于项目中所有模块的构建配置。

默认情况下，顶层 build 文件使用 `buildscript` 代码块定义项目中所有模块共用的 **Gradle 代码库**和**依赖项**。

```kotlin
buildscript { // gradle脚本执行所需依赖
    repositories { // 配置远程仓库
        google() // 引用google上的开源项目
        mavenCentral() // 引用 jcenter上的开源项目，现已经替换为 mavenCenter
    }
    dependencies { // 配置构建工具
        classpath("com.android.tools.build:gradle:7.0.2") 
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:1.6.0-M1") 
    }
}

// 运行gradle clean时，执行此处定义的task任务。
// 该任务继承自Delete，删除根目录中的build目录。
// 相当于执行Delete.delete(rootProject.buildDir)。
tasks.register("Delete", Delete::class) { 
    delete(rootProject.buildDir)
}
```

#### 配置项目全局属性

项目包含多个模块时，可以在文件中添加公用配置。

```kotlin
buildscript {...}
allprojects {...}

// 使用map
extra["compileSdkVersion"] = 28
// You can also create properties to specify versions for dependencies.
// Having consistent versions between modules can avoid conflicts with behavior.
extra["supportLibVersion"] = "28.0.0"

// 或者使用 Kotlin 委托 https://www.runoob.com/kotlin/kotlin-delegated.html
val compileSdkVersion by extra(31)
val supportLibVersion by extra("28.0.0")
```

在模块中使用以下代码引用即可：

```kotlin
val sdkVersion: Int by rootProject.extra
...
compileSdkVersion(sdkVersion)
```

> **注意**：虽然 Gradle 可让您在模块级别定义项目全局属性，但您应避免这样做，因为这样会导致共享这些属性的模块相互结合。[模块结合](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:decoupled_projects)使得以后将模块作为独立项目导出更加困难，并实际妨碍 Gradle 利用[并行项目执行](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:parallel_execution) 加快多模块构建。

### 模块 Build 文件

模块级 `build.gradle` 文件位于每个 `project/module/` 目录下，用于为其所在的特定模块配置构建设置。您可以通过配置这些 build 设置提供自定义打包选项（如额外的 build 类型和产品变种），以及替换 `main/` 应用清单或顶层 `build.gradle` 文件中的设置。

```kotlin
//声明是Android程序，
//com.android.application 表示这是一个应用程序模块
//com.android.library 标识这是一个库模块
plugins {
    id("com.android.application")
    id("kotlin-android")
}

// 上面全局配置的引用
val androidTargetSdkVersion: Int by rootProject.extra

// 配置项目构建的各种属性
android {
    compileSdk = androidCompileSdkVersion //设置编译时用的Android版本
    ...

    defaultConfig { // app 相关
        applicationId = defaultManagerPackageName // 包名
        minSdk = androidMinSdkVersion // 最低兼容版本
        targetSdk = androidTargetSdkVersion // 目标安卓版本
        versionCode = verCode // 版本号
        versionName = verName // 版本名称
        //若使用AndroidJUnitRunner进行单元测试
        //testInstrumentationRunner = "android.support.test.runner.AndroidJUnitRunner" 
    }
	//指定生成安装文件的主要配置，一般包含两个子闭包:
    //一个是debug闭包，用于指定生成测试版安装文件的配置，可以忽略不写；
    //另一个是release闭包，用于指定生成正式版安装文件的配置。
    buildTypes {
        release { // 一般使用如下两项
            isMinifyEnabled = false //是否对代码进行混淆
            proguardFiles("proguard-rules.pro") //指定混淆的规则文件
        }
    }
    compileOptions {
        sourceCompatibility(androidSourceCompatibility)
        targetCompatibility(androidTargetCompatibility)
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
    
    packagingOptions{//打包时的相关配置
        //这个是在同时使用butterknife、dagger2做的一个处理。同理，遇到类似的问题，只要根据gradle的提示，做类似处理即可。
        exclude 'META-INF/services/javax.annotation.processing.Processor'
    }

}

dependencies { // 项目的依赖
    // Dependency on a local library module
    implementation project(":mylibrary")
    // Dependency on local binaries
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    // Dependency on a remote binary
    implementation 'com.example.android:app-magic:12.3'
    ...
}

configurations.all {
    exclude(group = "androidx.appcompat", module = "appcompat")
}
```

### 更多模块配置项

#### [设置应用 ID](https://developer.android.com/studio/build/application-id)

#### [添加构建依赖项](https://developer.android.com/studio/build/dependencies)

#### [Android Gradle 插件可以使用的原生依赖项](https://developer.android.com/studio/build/native-dependencies)

#### [优化构建速度](https://developer.android.com/studio/build/optimize-your-build)

#### [排查构建性能问题](https://developer.android.com/studio/build/build-analyzer)

#### [分析构建性能](https://developer.android.com/studio/build/profile-your-build)

#### [配置 build 变体](https://developer.android.com/studio/build/build-variants)

#### [构建多个 APK](https://developer.android.com/studio/build/configure-apk-splits)

#### [合并多个清单文件](https://developer.android.com/studio/build/manifest-merge)

#### [将构建变量注入清单](https://developer.android.com/studio/build/manifest-build-variables)

#### [缩减、混淆处理和优化应用](https://developer.android.com/studio/build/shrink-code)

#### [为方法数超过 64K 的应用启用 MultiDex](https://developer.android.com/studio/build/multidex)

#### [使用 APK 分析器分析您的 build](https://developer.android.com/studio/build/apk-analyzer)

#### [使用 Maven Publish 插件](https://developer.android.com/studio/build/maven-publish-plugin)

#### [Gradle 提示与诀窍](https://developer.android.com/studio/build/gradle-tips)

#### [将构建配置从 Groovy 迁移到 KTS](https://developer.android.com/studio/build/migrate-to-kts)
