---
layout: post
title: Android Studio导入framework.jar等系统jar包方式
description: Android Studio导入framework.jar等系统jar包方式
keywords: Android, framework
---

### 场景说明

开发过程中需要依赖framework中隐藏方法的时候，可以导入framework.jar，
在编译的时候依赖framework，打包的时候不打进去，所以就需要了解Android studio中如何导入framework.jar包及其他系统jar包。

### 具体步骤
1. 拷贝framwork.jar 到项目的/libs 文件夹中
2. 修改app/build.gradle 文件，将该jar包加入依赖包列表中,注意需用的动作是 compileOnly。multiDexEnabled 添加分包支持。
```

android {
    compileSdkVersion 25
    defaultConfig {
        minSdkVersion 19
        targetSdkVersion 25
        flavorDimensions "armeabi-v7a"
        multiDexEnabled true // 打开这个选项

        ndk {
            abiFilters 'armeabi'
        }
    }
}

dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    compileOnly files('libs/framework.jar')
}
```
3. 修改project的build.gradle文件

```
    gradle.projectsEvaluated {
        tasks.withType(JavaCompile) {
            options.compilerArgs.add('-Xbootclasspath/p:app/libs/framework.jar')
        }
    }
```

4. application 的初始化中添加对multiDexEnabled支持，添加attachBaseContext的重载实现。

```
 @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        MultiDex.install(base);
    }
```
