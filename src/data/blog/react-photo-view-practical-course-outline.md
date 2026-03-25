---
author: Axton
pubDatetime: 2026-03-25T15:22:00Z
modDatetime: 2026-03-25T15:22:00Z
title: react-photo-view 实战课程大纲
featured: false
draft: false
tags:
  - docs
description:
  用 1-2 天时间，从零到一实现一个完整的图片预览组件，特别掌握移动端长图预览功能
---

# react-photo-view 实战课程大纲

**目标**：用 1-2 天时间，从零到一实现一个完整的图片预览组件，特别掌握移动端长图预览功能

**前置知识**：熟悉 React 基本语法、了解 TypeScript 基础

---

## 课程安排

| 天数  | 主题               | 时长     | 产出             |
| ----- | ------------------ | -------- | ---------------- |
| Day 1 | 基础架构与核心组件 | 6-8 小时 | 实现基础图片预览 |
| Day 2 | 手势交互与动画     | 6-8 小时 | 实现完整功能     |

---

## Day 1：基础架构与核心组件

### 上午：环境准备与基础概念（2小时）

#### 第1课：项目初始化与环境搭建 [30分钟]

- [ ] 创建 React + TypeScript 项目
- [ ] 配置开发环境
- [ ] 安装必要依赖

**实践任务**：初始化项目，安装 react、typescript、less

```bash
npx create-react-app my-photo-view --template typescript
# 或
npm create vite@latest my-photo-view -- --template react-ts
```

---

#### 第2课：理解组件架构 [30分钟]

- [ ] 分析 react-photo-view 的整体架构
- [ ] 理解 Provider / View / Slider 三大组件关系
- [ ] 理解数据流方向

**核心概念**：

```
┌─────────────────────────────────────────────────┐
│                  PhotoProvider                   │
│  ┌─────────────────────────────────────────┐   │
│  │            PhotoContext                  │   │
│  │  - images[]                             │   │
│  │  - visible                              │   │
│  │  - index                                │   │
│  │  - show()/update()/remove()            │   │
│  └─────────────────────────────────────────┘   │
│                         │                        │
│         ┌──────────────┴──────────────┐         │
│         ▼                              ▼         │
│  ┌─────────────┐              ┌─────────────┐    │
│  │  PhotoView  │              │ PhotoSlider │    │
│  │  (单图组件)  │              │  (预览主组件) │    │
│  └─────────────┘              └─────────────┘    │
└─────────────────────────────────────────────────┘
```

---

#### 第3课：TypeScript 类型系统实战 [60分钟]

- [ ] 类型定义基础（interface、type）
- [ ] 泛型的使用
- [ ] React 类型（FC、Ref、Props）

**实践任务**：定义图片预览组件的完整类型

```typescript
// 定义资源类型
interface PhotoItem {
  key: string | number;
  src?: string;
  width?: number;
  height?: number;
  render?: (props: RenderProps) => React.ReactNode;
}

// 定义 Provider Props
interface PhotoProviderProps {
  children: React.ReactNode;
  images?: PhotoItem[];
  onIndexChange?: (index: number) => void;
  onVisibleChange?: (visible: boolean) => void;
}

// 定义 Context 类型
interface PhotoContextType {
  visible: boolean;
  index: number;
  images: PhotoItem[];
  show: (index: number) => void;
  hide: () => void;
}
```

---

### 下午：核心组件实现（4-6小时）

#### 第4课：实现 PhotoContext [60分钟]

- [ ] 创建 Context
- [ ] 定义 Context 类型
- [ ] 理解 Context 的作用

**实践任务**：创建 photo-context.ts

```typescript
import { createContext } from 'react';

export interface PhotoItem {
  key: string | number;
  src?: string;
  width?: number;
  height?: number;
}

export interface PhotoContextType {
  visible: boolean;
  index: number;
  images: PhotoItem[];
  show: (index: number) => void;
  hide: () => void;
}

const PhotoContext = createContext<PhotoContextType | undefined>(undefined!);

export default PhotoContext;
```

**产出**：完成 `photo-context.ts`

---

#### 第5课：实现 PhotoProvider 状态管理 [90分钟]

- [ ] 管理 images、visible、index 状态
- [ ] 实现 show/hide/changeIndex 方法
- [ ] 使用 useReducer 或 useSetState
- [ ] 提供 Context Provider

**实践任务**：实现 PhotoProvider.tsx

```typescript
import { useState, useCallback, useMemo, createContext } from 'react';
import PhotoContext from './photo-context';

export default function PhotoProvider({ children }) {
  const [visible, setVisible] = useState(false);
  const [index, setIndex] = useState(0);
  const [images, setImages] = useState([]);

  const show = useCallback((index: number) => {
    setIndex(index);
    setVisible(true);
  }, []);

  const hide = useCallback(() => {
    setVisible(false);
  }, []);

  const value = useMemo(() => ({
    visible,
    index,
    images,
    show,
    hide,
  }), [visible, index, images, show, hide]);

  return (
    <PhotoContext.Provider value={value}>
      {children}
    </PhotoContext.Provider>
  );
}
```

**产出**：完成 `PhotoProvider.tsx`

---

#### 第6课：实现 PhotoView 单图组件 [60分钟]

- [ ] 接收 src、children 等 props
- [ ] 注册到 Provider
- [ ] 点击触发预览
- [ ] 克隆子组件传递事件

**实践任务**：实现 PhotoView.tsx

```typescript
import { useContext, useEffect, useRef } from 'react';
import PhotoContext from './photo-context';

export default function PhotoView({ src, children }) {
  const { show, images, setImages } = useContext(PhotoContext);
  const key = useRef(Symbol('photo')).current;

  useEffect(() => {
    // 注册到 Provider
    setImages((prev) => [...prev, { key, src }]);
    return () => {
      // 清理
      setImages((prev) => prev.filter((img) => img.key !== key));
    };
  }, [src]);

  const handleClick = () => {
    const index = images.findIndex((img) => img.key === key);
    show(index);
  };

  if (children) {
    return cloneElement(children, { onClick: handleClick });
  }
  return null;
}
```

**产出**：完成 `PhotoView.tsx`

---

#### 第7课：实现基础 PhotoSlider [90分钟]

- [ ] 接收 images、visible、index
- [ ] 使用 Portal 渲染
- [ ] 实现图片切换（点击按钮）
- [ ] 实现关闭功能
- [ ] 添加遮罩层

**实践任务**：实现基础的 PhotoSlider

```typescript
import { useEffect, usePortal } from 'react';

export default function PhotoSlider({ visible, images, index, onClose }) {
  if (!visible) return null;

  // 使用 Portal 渲染到 body
  return createPortal(
    <div className="photo-slider-overlay" onClick={onClose}>
      <div className="photo-slider-content" onClick={e => e.stopPropagation()}>
        <img src={images[index]?.src} alt="" />

        {/* 关闭按钮 */}
        <button onClick={onClose}>✕</button>

        {/* 切换按钮 */}
        {index > 0 && <button onClick={prev}>◀</button>}
        {index < images.length - 1 && <button onClick={next}>▶</button>}
      </div>
    </div>,
    document.body
  );
}
```

**产出**：完成基础版 `PhotoSlider.tsx`

---

### 晚上：回顾与练习（1小时）

#### 第8课：Day1 总结与练习 [60分钟]

- [ ] 回顾今天实现的代码
- [ ] 确保基础功能正常运行
- [ ] 练习：添加多张图片支持

**检查清单**：

- [ ] 点击图片可以打开预览
- [ ] 点击遮罩可以关闭预览
- [ ] 可以切换上一张/下一张图片

---

## Day 2：手势交互与动画

### 上午：触摸事件处理（3小时）

#### 第9课：触摸事件基础 [60分钟]

- [ ] 理解 Touch Events（touchstart、touchmove、touchend）
- [ ] 掌握事件对象（touches、changedTouches）
- [ ] 理解触摸坐标获取

**核心概念**：

```typescript
// 触摸开始
const handleTouchStart = (e: TouchEvent) => {
  const touch = e.touches[0];
  const startX = touch.clientX;
  const startY = touch.clientY;
};

// 触摸移动
const handleTouchMove = (e: TouchEvent) => {
  const touch = e.touches[0];
  const deltaX = touch.clientX - startX;
  const deltaY = touch.clientY - startY;
};

// 触摸结束
const handleTouchEnd = (e: TouchEvent) => {
  // 判断滑动方向和距离
};
```

---

#### 第10课：实现基础滑动切换 [90分钟]

- [ ] 监听触摸事件
- [ ] 计算滑动距离
- [ ] 判断滑动方向
- [ ] 触发图片切换

**实践任务**：给 PhotoSlider 添加滑动切换

```typescript
function useSwipe(onSwipeLeft, onSwipeRight) {
  const [startX, setStartX] = useState(0);
  const [startY, setStartY] = useState(0);

  const handleTouchStart = (e) => {
    setStartX(e.touches[0].clientX);
    setStartY(e.touches[0].clientY);
  };

  const handleTouchEnd = (e) => {
    const endX = e.changedTouches[0].clientX;
    const endY = e.changedTouches[0].clientY;
    const deltaX = endX - startX;
    const deltaY = endY - startY;

    // 判断是水平滑动还是垂直滑动
    if (Math.abs(deltaX) > Math.abs(deltaY)) {
      // 水平滑动
      if (Math.abs(deltaX) > 50) {
        // 阈值
        if (deltaX > 0) onSwipeRight();
        else onSwipeLeft();
      }
    }
  };

  return { handleTouchStart, handleTouchEnd };
}
```

**产出**：支持滑动切换的 PhotoSlider

---

#### 第11课：长图预览核心原理 [60分钟] **⭐重点**

- [ ] 理解长图与普通图片的区别
- [ ] 掌握 scrollWidth / scrollHeight
- [ ] 理解 translateY 滚动原理

**核心概念**：

```
┌────────────────────────────┐
│      视口 (viewport)        │
│  ┌──────────────────────┐  │
│  │                      │  │
│  │     长图内容          │  │  ◀── 可滚动
│  │                      │  │
│  │                      │  │
│  └──────────────────────┘  │
└────────────────────────────┘

长图滚动原理：
- 图片高度 > 视口高度时，允许滚动
- translateY 控制滚动位置
- scrollY = translateY 的绝对值
```

**长图 vs 普通图**：

| 特性 | 普通图片     | 长图                    |
| ---- | ------------ | ----------------------- |
| 尺寸 | 适应屏幕     | 原尺寸显示              |
| 交互 | 左右滑动切换 | 上下滚动查看            |
| 判断 | 宽高比       | scrollHeight > 视口高度 |

---

### 下午：完整手势交互实现（4小时）

#### 第12课：实现长图滚动预览 [90分钟] **⭐重点**

- [ ] 检测图片是否超出视口
- [ ] 实现 translateY 滚动
- [ ] 处理边界情况

**实践任务**：实现长图滚动

```typescript
function LongImageViewer({ src }) {
  const [translateY, setTranslateY] = useState(0);
  const [startY, setStartY] = useState(0);
  const imgRef = useRef<HTMLImageElement>(null);

  // 检测是否需要滚动
  const needsScroll = imgRef.current?.scrollHeight > window.innerHeight;

  const handleTouchMove = (e) => {
    if (!needsScroll) return; // 不需要滚动时，交还给父组件处理滑动切换

    const deltaY = e.touches[0].clientY - startY;
    const newTranslateY = translateY + deltaY;

    // 边界检查
    const maxTranslate = 0;
    const minTranslate = -(imgRef.current!.scrollHeight - window.innerHeight);

    if (newTranslateY > maxTranslate) {
      // 超出顶部，允许拖动但有阻力
      setTranslateY(newTranslateY * 0.3);
    } else if (newTranslateY < minTranslate) {
      // 超出底部，允许拖动但有阻力
      setTranslateY(minTranslate + (newTranslateY - minTranslate) * 0.3);
    } else {
      setTranslateY(newTranslateY);
    }
  };

  return (
    <img
      ref={imgRef}
      src={src}
      style={{
        transform: `translateY(${translateY}px)`,
        transition: isMoving ? 'none' : 'transform 0.3s'
      }}
    />
  );
}
```

**产出**：支持长图滚动的 PhotoSlider

---

#### 第13课：实现惯性滚动 [90分钟] **⭐重点**

- [ ] 计算滑动速度
- [ ] 实现惯性动画
- [ ] 添加边缘回弹效果

**实践任务**：实现物理滚动效果

```typescript
function useInertialScroll(translateY, setTranslateY, minY, maxY) {
  const lastY = useRef(0);
  const lastTime = useRef(0);

  const handleTouchEnd = (velocityY) => {
    // 速度超过阈值，触发惯性滚动
    if (Math.abs(velocityY) > 0.5) {
      animate({
        from: translateY,
        to: translateY + velocityY * 1000, // 根据速度计算目标位置
        duration: 500,
        easing: 'cubic-bezier(0.25, 0.8, 0.25, 1)',
        onUpdate: (value) => {
          // 边界限制与回弹
          if (value > maxY) {
            value = maxY + (value - maxY) * 0.3;
          } else if (value < minY) {
            value = minY + (value - minY) * 0.3;
          }
          setTranslateY(value);
        },
      });
    }
  };

  return { handleTouchEnd };
}
```

**产出**：带惯性滚动的长图预览

---

#### 第14课：实现图片缩放 [60分钟]

- [ ] 理解 scale 变换
- [ ] 实现双指缩放
- [ ] 处理缩放边界
- [ ] 缩放与滚动的冲突处理

**缩放核心逻辑**：

```typescript
function usePinchZoom() {
  const [scale, setScale] = useState(1);
  const [startDistance, setStartDistance] = useState(0);

  const handleTouchMove = (e) => {
    if (e.touches.length === 2) {
      // 计算双指距离
      const distance = Math.hypot(
        e.touches[0].clientX - e.touches[1].clientX,
        e.touches[0].clientY - e.touches[1].clientY,
      );

      if (startDistance === 0) {
        setStartDistance(distance);
      } else {
        const newScale = (distance / startDistance) * scale;
        // 限制缩放范围
        setScale(Math.min(Math.max(newScale, 1), 4));
      }
    }
  };

  return { scale, handleTouchMove };
}
```

---

### 晚上：动画与完善（2小时）

#### 第15课：入场/退场动画 [60分钟]

- [ ] 实现 Fade 动画（透明度变化）
- [ ] 实现 Scale 动画（缩放变化）
- [ ] 使用 CSS transition

```css
.photo-slider-overlay {
  animation: fadeIn 0.3s ease-out;
}

.photo-slider-content img {
  animation: scaleIn 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes scaleIn {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
```

---

#### 第16课：来源动画（FLIP）[60分钟] **⭐进阶**

- [ ] 理解 FLIP 原理
- [ ] 获取来源元素位置
- [ ] 实现动画过渡

**FLIP 核心**：

```
F - First:   记录初始位置和尺寸
L - Last:    记录最终位置和尺寸
I - Invert:  计算差异，应用变换
P - Play:    播放动画，移除变换
```

---

#### 第17课：Day2 总结与完整整合 [60分钟]

- [ ] 整合所有功能模块
- [ ] 测试各种边界情况
- [ ] 优化代码结构

**最终检查清单**：

- [ ] 普通图片支持滑动切换
- [ ] 长图支持上下滚动
- [ ] 支持双指缩放
- [ ] 惯性滚动效果自然
- [ ] 边缘回弹效果自然
- [ ] 入场/退场动画流畅

---

## 完整功能清单

完成课程后，你将实现一个功能完整的图片预览组件：

### 基础功能

- [x] 点击图片打开预览
- [x] 点击遮罩关闭预览
- [x] 切换上一张/下一张
- [x] 循环预览支持
- [x] 键盘左右键切换（桌面端）

### 手势交互

- [x] 左右滑动切换图片（普通图片）
- [x] 上下滚动查看长图（长图）
- [x] 双指缩放图片
- [x] 惯性滚动效果
- [x] 边缘回弹效果

### 动画效果

- [x] 入场/退场动画
- [x] 切换过渡动画
- [x] 缩放过渡动画
- [x] 边缘回弹动画

### 高级功能

- [x] 自定义渲染（支持 video 等）
- [x] 自定义工具栏
- [x] Loading 状态
- [x] 错误处理

---

## 项目结构

完成课程后，你的项目结构如下：

```
src/
├── photo-context.ts       # Context 定义
├── PhotoProvider.tsx       # 状态管理
├── PhotoView.tsx          # 单图组件
├── PhotoSlider.tsx        # 预览主组件
├── hooks/
│   ├── useSwipe.ts        # 滑动切换
│   ├── useScroll.ts       # 滚动处理
│   ├── useZoom.ts         # 缩放处理
│   ├── useInertial.ts     # 惯性滚动
│   └── useAnimation.ts    # 动画控制
├── utils/
│   ├── detectImageType.ts  # 检测图片类型
│   └── math.ts            # 数学计算
└── styles/
    └── PhotoSlider.less   # 样式
```

---

## 学习资源

### 推荐阅读

- [MDN Touch Events](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)
- [CSS Transform](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)
- [FLIP Animations](https://aerotwist.com/blog/flip-your-animations/)

### 实战参考

- react-photo-view 源码
- react-image-lightbox 源码
- photoswipe 源码

---

## 常见问题

### Q1: 如何判断是长图还是普通图？

```typescript
function detectImageType(imgHeight: number, viewportHeight: number): 'long' | 'normal' {
  return imgHeight > viewportHeight ? 'long' : 'normal';
}
```

### Q2: 缩放和滚动冲突如何处理？

```typescript
// 缩放时禁用滚动
// 滚动时禁用缩放
// 使用状态控制
const [mode, setMode] = useState<'scroll' | 'zoom' | 'none'>('none');

// 单指移动 → 滚动模式
// 双指移动 → 缩放模式
if (touches.length === 1) setMode('scroll');
if (touches.length === 2) setMode('zoom');
```

### Q3: 性能优化怎么做？

- 使用 `transform` 而非 `top/left`
- 使用 `will-change: transform`
- 避免在 touchmove 中触发重渲染
- 使用 `requestAnimationFrame`

---

## 课程总结

### 核心技能

| 技能            | 掌握程度 |
| --------------- | -------- |
| React Context   | ✅ 熟练  |
| TypeScript 类型 | ✅ 熟练  |
| Touch Events    | ✅ 熟练  |
| CSS Transform   | ✅ 熟练  |
| 物理滚动算法    | ✅ 掌握  |
| FLIP 动画       | ✅ 了解  |

### 下一步

1. 添加更多功能（旋转、下载）
2. 优化性能和体验
3. 编写测试用例
4. 发布到 npm

---

_课程结束，祝你开发愉快！_
