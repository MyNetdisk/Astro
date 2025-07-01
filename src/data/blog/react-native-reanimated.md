---
author: Axton
pubDatetime: 2025-07-01T15:22:00Z
modDatetime: 2025-07-01T15:22:00Z
title: 解决react-native-reanimated使用过程报错
featured: false
draft: false
tags:
  - React Native
description:
  "ReferenceError: Property '_WORKLET' doesn't exist, js engine: hermes"
---

解决报错：ReferenceError: Property '_WORKLET' doesn't exist, js engine: hermes

---

### 🐞 报错信息

> ReferenceError: Property '_WORKLET' doesn't exist, js engine: hermes

### 🛠️ 开发环境

* Windows 11 OS
* react: "19.0.0"
* react-native-svg: "^15.11.2"
* react-native: "0.78.1"

### ✅ 解决方案（逐步）：

#### ✅ 1. 确保你 Babel 配置支持 Reanimated（这是**最关键的一步**）

编辑根目录的 **`babel.config.js`**，必须在 plugins 里加上：
```js
module.exports = {
  presets: ['module:metro-react-native-babel-preset'],
  plugins: [
    'react-native-reanimated/plugin', // ⬅️ 必须是 plugins 的最后一个
  ],
};
```

#### ✅ 2. 确保你不是在使用旧的写法

例如旧写法中：
```ts
const animatedProps = useAnimatedProps(() => ({
  strokeDashoffset: progress.value, // ❌ v3+ 中，如果你没配置好，就可能 _WORKLET 报错
}));
```
在 Reanimated v3 中你其实不需要 `'_WORKLET'` 注释，**也不需要做额外的配置**，只要 babel 正确。

#### 🧼 3. 清缓存 + 重启构建（这是**必须的**）
```bash
npx react-native start --reset-cache
npx react-native run-android
```