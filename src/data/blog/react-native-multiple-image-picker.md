---
author: Axton
pubDatetime: 2025-07-02T15:22:00Z
modDatetime: 2025-07-02T15:22:00Z
title: 解决@baronha/react-native-multiple-image-picker使用过程报错
featured: false
draft: false
tags:
  - React Native
description:
  "Execution failed for task ':baronha_react-native-multiple-image-picker:buildCMakeDebug[arm64-v8a]'. "
---

## 🐞 报错信息

> Execution failed for task ':baronha_react-native-multiple-image-picker:buildCMakeDebug[arm64-v8a]'.

## 🔍问题分析

1. **核心错误**: `ninja: error: manifest 'build.ninja' still dirty after 100 tries`
2. **CMake 重复运行**: 日志显示 CMake 配置被重复执行了很多次
3. **构建系统问题**: C++ 构建系统在执行时失败
## 🛠️ 开发环境

- Windows 11 OS
- react: "19.0.0",
- react-native: "0.79.2"
- @baronha/react-native-multiple-image-picker: "^2.2.3",
- react-native-nitro-modules: "^0.25.2",

### ✅ 解决方案（逐步）：

### 检查 NDK 和 CMake 版本
在 `android/build.gradle` 中确保使用正确的版本：
```gradle
buildscript {

    ext {

        ndkVersion = "26.1.10909125" // 这里是和包项目@baronha/react-native-multiple-image-picker中example的配置保持一致。
        
        cmakeVersion = "3.18.1"  // 添加这行, 这是最关键的一步🚀，报错时这里是"3.22.1"

    }

}
```
在`android/app/build.gradle`中确保进行下面的配置
```
android {
    ...

    defaultConfig {

        applicationId "com.awesomeproject"

        minSdkVersion rootProject.ext.minSdkVersion

        targetSdkVersion rootProject.ext.targetSdkVersion

        versionCode 1

        versionName "1.0"

  

        // 添加 CMake 配置, 这是最关键的一步🚀

        externalNativeBuild {

            cmake {

                version rootProject.ext.cmakeVersion

                arguments "-DANDROID_STL=c++_shared"

                cppFlags "-std=c++17"

            }

        }

    }
    ...

}
```
### 在`android/gradle.properties`中去掉模拟器支持：
```properties
reactNativeArchitectures=arm64-v8a // 仅保留arm64-v8a架构
```

### 打包发布时或许需要在proguard-rules.pro文件中配置不需要代码混淆
```pro
# Add project specific ProGuard rules here.
# By default, the flags in this file are appended to flags specified
# in /usr/local/Cellar/android-sdk/24.3.3/tools/proguard/proguard-android.txt
# You can edit the include path and order by changing the proguardFiles
# directive in build.gradle.
#
# For more details, see
#   http://developer.android.com/guide/developing/tools/proguard.html

# Add any project specific keep options here:

-keep public class com.margelo.nitro.multipleimagepicker.* {*;}
-keep public class com.margelo.nitro.multipleimagepicker.** {*;}
```

### 手动清理构建缓存
```bash
rm -rf android/build
rm -rf android/.gradle
rm -rf android/app/build
rm -rf android/app/.cxx

rm -rf node_modules
```

### 重新构建
```bash
npm install # 或者使用 yarn install
# 清理 React Native 缓存 
npx react-native start --reset-cache
npx react-native run-android
```
### 如果还是有问题
由于你使用的是 NDK 26.1.10909125（比较新），建议尝试：

1. **先用 CMake 3.28.0**（与新 NDK 更兼容）
2. **如果不行，降级到 CMake 3.18.1**
3. **最后考虑降级 NDK 到 23.1.7779620**

你的版本组合比较新，可能存在一些兼容性问题。建议先尝试添加 `cmakeVersion = "3.18.1"`，这是一个经过广泛测试的稳定版本。
