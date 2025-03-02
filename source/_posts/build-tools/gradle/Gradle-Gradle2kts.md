---
title: 从Groovy迁移到KTS
date: 2021-09-25 22:29
categories: [Build Tools, Gradle]
tag: Gradle
toc: true
updated: 2021-09-25 22:29
references:
  - title: 将构建配置从 Groovy 迁移到 KTS  | Android 开发者  | Android Developers
    url: https://developer.android.com/studio/build/migrate-to-kts?hl=zh-cn
  - title: 快速迁移 Gradle 脚本至 KTS - 知乎 (zhihu.com)
    url: https://zhuanlan.zhihu.com/p/365834468
---

在 Android Studio 中将 Gradle 配置从 Groovy 迁移到 KTS，即用 Kotlin 代替 Groovy 编写 Gradle 脚本。

<!-- more -->

## 转换规则

- Groovy 允许使用单引号来定义字符串，而 Kotlin 则要求使用双引号：

	即 ` include ':app' -> include ":app" // '(.*?[^\\])' 正则替换 "$1"  `

- 在 Groovy 中使用 `$` 前缀来表示基于句点表达式的字符串插值，但在 Kotlin 中需要使用大括号括住整个变量：

	即 `myRootDirectory = "$project.rootDir/tools"` 改为：`myRootDirectory = "${project.rootDir}/tools"`

- 显式和隐式 `buildTypes`：在 KTS 中，仅 `debug` 和 `release` `buildTypes` 是隐式提供的，而 `staging` 则必须手动创建

	```kotlin
	buildTypes
	  getByName("debug") {
	    ...
	  }
	  getByName("release") {
	    ...
	  }
	  create("staging") {
	    ...
	  }
	```

- 使用 `plugins` 代码块：给方法调用加上括号

	```kotlin
	// (\w+) (([^=\{\s]+)(.*)) 正则替换 $1($2)
	plugins {
	    id("com.android.application")
	    id("kotlin-android")
	    id("kotlin-kapt")
	    id("androidx.navigation.safeargs.kotlin")
	}
	```

> 更多请参考 [Gradle | Kotlin (kotlinlang.org)](https://kotlinlang.org/docs/gradle.html)

## 逐个迁移

按上述转换规则逐个迁移以下文件，每迁移一个文件，都 sync 一遍查看是否出现问题。

**下列每个迁移前，先把文件扩展名添加为 `*.gradle.kts`**

### 迁移 `setting.gradle`

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "your app name"
include(":app")
```

### 迁移项目的 `build.gradle`

```kotlin
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath("com.android.tools.build:gradle:7.0.2")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:1.5.20")
    }
}

tasks.register("Delete", Delete::class) {
    delete(rootProject.buildDir)
}
```

### 迁移模块的 `build.gradle`

```kotlin
plugins {
    id("com.android.application")
    id("kotlin-android")
}

android {
    compileSdk = 30

    defaultConfig {
        applicationId = "github.sukieva.hhuer"
        minSdk = 24
        targetSdk = 30
        versionCode = 1
        versionName = "1.0"

    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles("proguard-rules.pro")
        }
    }
    compileOptions {
        sourceCompatibility(JavaVersion.VERSION_1_8)
        targetCompatibility(JavaVersion.VERSION_1_8)
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
}

dependencies {

    implementation("androidx.core:core-ktx:1.6.0")
    implementation("androidx.appcompat:appcompat:1.3.1")
    implementation("com.google.android.material:material:1.4.0")
    implementation("androidx.constraintlayout:constraintlayout:2.1.0")
    testImplementation("junit:junit:4.+")
    androidTestImplementation("androidx.test.ext:junit:1.1.3")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.4.0")
}
```

### 代码优化

可以在项目的 `build.gradle.kts` 里添加定义属性，在其他配置文件可直接访问。

```kotlin
val defaultManagerPackageName by extra("github.sukieva.hhuer")
val verCode by extra(20210924)
val verName by extra("1.0")
val androidTargetSdkVersion by extra(31)
val androidMinSdkVersion by extra(27)
val androidBuildToolsVersion by extra("31.0.0")
val androidCompileSdkVersion by extra(31)
val androidCompileNdkVersion by extra("23.0.7599858")
val androidSourceCompatibility by extra(JavaVersion.VERSION_11)
val androidTargetCompatibility by extra(JavaVersion.VERSION_11)
```

## 可能出现的问题

### *code Insight unavailable*

因为期间重启过几次 Android Studio 不太确定怎么排除的，可能切换 kotlin 到 dev 版能解决此问题。
