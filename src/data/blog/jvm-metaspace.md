---
author: Axton
pubDatetime: 2025-07-09T15:22:00Z
modDatetime: 2025-07-09T15:22:00Z
title: JVM Metaspace 内存不足导致的构建失败
featured: false
draft: false
tags:
  - React Native
description:
  "> Task :app:minifyReleaseWithR8 FAILED ERROR: R8: java.lang.OutOfMemoryError: Metaspace"
---

你遇到的错误：

```
> Task :app:minifyReleaseWithR8 FAILED  
ERROR: R8: java.lang.OutOfMemoryError: Metaspace
```

以及多个模块（如 `react-native-permissions`, `react-native-pager-view`）出现的：

```
> A failure occurred while executing com.android.build.gradle.internal.lint.AndroidLintWorkAction        
   > Metaspace
```

### ✅ 问题分析：

这是典型的 **JVM Metaspace 内存不足导致的构建失败**，出现在：

* **R8 压缩和混淆**阶段
* **Lint 分析**阶段

---

## 🔧 解决方法：

### ✅ 方法一：增加 JVM 的 Metaspace 大小（推荐）

编辑 `android/gradle.properties` 文件，添加或修改：

```properties
org.gradle.jvmargs=-Xmx4g -XX:MaxMetaspaceSize=1024m
```

解释：

* `-Xmx4g`：最大堆内存，建议设置至少为 `2g` 或 `4g`
* `-XX:MaxMetaspaceSize=1024m`：设置最大 metaspace 为 1024MB（可根据你内存适当调整）

---

### ✅ 方法二：禁用 Lint Vital 分析（可选）

如果你只是想构建 Release 包而不需要发布到 Google Play，可以在 `android/app/build.gradle` 中禁用 Lint Vital：

```groovy
android {
  lintOptions {
    checkReleaseBuilds false
    abortOnError false
  }
}
```

或者在 `build.gradle` 中针对 `lintVitalAnalyzeRelease` 任务跳过（仅限调试用途）：

```bash
./gradlew assembleRelease -x lintVitalAnalyzeRelease
```

---

### ✅ 方法三：关闭 R8（不推荐，除非你明确知道你不需要混淆）

在 `gradle.properties` 中添加：

```properties
android.enableR8=false
```

⚠️ **注意：R8 是默认推荐的代码压缩混淆工具，禁用它会使你丢失优化和混淆，尽量避免禁用。**

---

### ✅ 方法四：升级 Gradle 或 R8 版本（如果你内存够但还是失败）

某些 R8 版本有 **内存泄漏或 bug**，可以尝试升级：

* 升级 Gradle Wrapper 到 8.x（你当前提示使用了 `8.12`，兼容性 OK）
* 在 `android/build.gradle` 中使用最新的 AGP（Android Gradle Plugin）：

```gradle
classpath("com.android.tools.build:gradle:8.1.1")
```

* 使用更高版本的 R8（在 `dependencies` 中指定）：

```gradle
dependencies {
  // 可选：强制使用最新版本的 R8（一般不建议手动指定，Gradle/AGP 会管理）
}
```

---

## 📌 总结优先建议操作顺序：

1. 修改 `gradle.properties` 增加 Metaspace：

```properties
org.gradle.jvmargs=-Xmx4g -XX:MaxMetaspaceSize=1024m # 这一步解决了问题！！！
```

2. 如果还不行，再考虑关闭 lint 检查或调低 R8 优化级别。
