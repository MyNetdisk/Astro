---
author: Axton
pubDatetime: 2025-07-17T15:22:00Z
modDatetime: 2025-07-17T15:22:00Z
title: zustand和redux、mobx等库的区别？以及如何使用zustand？
featured: false
draft: false
tags:
  - zustand
  - redux
  - mobx
description:
  "Zustand 是当前非常流行的 React 状态管理库之一，简单、轻量、灵活，很多项目都在从 Redux 或 MobX 迁移到它。"
---

## Table of contents

## ✅ 一句话总结：

> **Zustand 是一个轻量级、无模板、Hooks 风格的状态管理库，适用于中小型 React 应用。**

---

## 🧠 它和 Redux / MobX 有什么区别？

|特性|Zustand|Redux|MobX|
|---|---|---|---|
|🧱 核心设计|Hook + Store 函数|Action + Reducer + Store|响应式（可观察）对象|
|📦 依赖复杂度|极低（无 Provider，无模板）|中等（需 middleware，Redux Toolkit）|中等（需 class 或 observable）|
|🧠 学习曲线|简单|偏陡（需要理解 reducer / action）|简单但略魔法|
|⛓️ 约束性强弱|弱（可随意组织 store）|强（必须使用规定架构）|中（受 MobX 语法限制）|
|🧩 推荐场景|中小型项目、组件状态共享|大型项目，需调试、日志、持久化等|小型项目或有响应式需求的|
|🛠 Devtools 支持|✅ 有（可选）|✅ 很强（Redux Devtools）|✅ 有|
|🔁 状态更新性能|非常快（浅比较）|快（但需要 connect / selector）|快（细粒度观察）|

---

## 🛠 如何使用 Zustand？

### 1️⃣ 安装

```bash
npm install zustand
# 或
yarn add zustand
```

---

### 2️⃣ 创建 Store

```ts
// store/useUserStore.ts
import { create } from 'zustand'

interface UserState {
  name: string
  setName: (name: string) => void
}

export const useUserStore = create<UserState>((set) => ({
  name: 'Tom',
  setName: (name) => set({ name }),
}))
```

---

### 3️⃣ 在组件中使用

```tsx
import React from 'react'
import { useUserStore } from '@/store/useUserStore'

export default function UserInfo() {
  const name = useUserStore((state) => state.name)
  const setName = useUserStore((state) => state.setName)

  return (
    <div>
      <h1>用户名：{name}</h1>
      <button onClick={() => setName('Alice')}>改名为 Alice</button>
    </div>
  )
}
```

---

## ✅ 使用技巧

### 🧩 多 store 分离管理（每个功能一个 store）

```ts
// store/useCounterStore.ts
export const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}))
```

---

### 🧠 Zustand 会做哪些优化？

- 🔁 自动浅比较，避免组件不必要的重渲染
    
- ✅ 与 React DevTools 兼容
    
- ✅ 支持异步逻辑
    
- ✅ 支持 selector、middleware（如持久化）
    
- ✅ 可以只监听状态的一部分（解耦）
    

---

## 🌟 状态持久化（可选）

```ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useAuthStore = create(
  persist(
    (set) => ({
      token: '',
      setToken: (token: string) => set({ token }),
    }),
    {
      name: 'auth-storage', // localStorage key
    }
  )
)
```

---

## 🚀 与 Redux 对比图示（概念）

||Zustand|Redux|
|---|---|---|
|写法|`useStore((state) => ...)`|`connect(mapStateToProps)`|
|状态结构|任意 JS 对象|单一 reducer 树|
|状态修改|直接写函数调用（set）|必须 dispatch action + reducer|
|初始化|create(fn)|configureStore()|

---

## 📌 总结：Zustand 适合你吗？

|如果你...|那就选择|
|---|---|
|想用最少代码管理状态|✅ Zustand|
|项目中状态不是很复杂|✅ Zustand|
|需要精细性能优化|✅ Zustand|
|状态逻辑需要很强组织性|Redux（搭配 Toolkit）可能更好|
|对响应式、自动依赖收集感兴趣|MobX 也可选|

---

## ✅ 示例项目结构建议（中大型项目）

```
src/
├── store/
│   ├── useUserStore.ts
│   ├── useAuthStore.ts
│   └── useSettingsStore.ts
├── components/
│   └── ...
├── pages/
│   └── ...
```
