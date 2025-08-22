---
author: Axton
pubDatetime: 2025-08-22T15:22:00Z
modDatetime: 2025-08-22T15:22:00Z
title: 使用 Flex 布局在不固定高度下实现可滚动列表的技巧
featured: false
draft: false
tags:
  - CSS
description:
  "使用 Flex 布局在不固定高度下实现可滚动列表的技巧"
---

# 使用 Flex 布局在不固定高度下实现可滚动列表的技巧

## 背景

在前端开发中，我们经常遇到这样的需求：

- 页面整体采用 **Flex 布局**；
    
- 中间某一部分内容需要根据父容器的剩余空间自适应高度；
    
- 这部分内容可能很多，要求 **超出容器时显示滚动条**；
    
- 外层容器没有固定高度，而是依赖父布局自适应。
    

常规的 `flex: 1` 配合 `overflow: auto` 在一些场景下可能无法生效，导致滚动条不出现。本文将介绍一种“**height: 0 + flex: 1**”的技巧，解决这一问题。

---

## 示例代码

```tsx
// 空间未选中文件/文件夹时的渲染逻辑
const renderMySpaceNotSelect = (isGroupSpaceRender = false) => {
  return (
    <div
      style={{
        flex: 1,
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'flex-start',
        paddingBottom: '16px',
      }}
    >
      {/* 渲染管理按钮（仅限个人空间） */}
      {!isGroupSpaceRender && isGroupSpace && renderManagementBtn()}

      {/* 标题区域 */}
      <div className={styles.rightSidebarCardTitle}>最近使用</div>

      {/* 列表区域（滚动容器） */}
      <div style={{ flex: 1, overflowY: 'auto' }}>
        {/* 
          核心技巧：
          - height: 0 是“欺骗”浏览器计算高度，让外层 flex 子项的高度按 0 处理
          - 这样 flex: 1 才能真正生效，分配到父容器剩余高度
          - 从而使 overflowY: auto 起效，列表内容超出时出现滚动条
        */}
        <div style={{ height: 0 }}>
          {renderRecentViewList([])}
        </div>
      </div>
    </div>
  );
};
```

---

## 技术要点

### 1. 为什么需要 `height: 0`？

- 在 **Flex 子项** 中，`flex: 1` 默认会让子元素 **尽可能填满父容器**；
    
- 如果子项内部有内容，浏览器会尝试把 **内容高度** 加到计算结果中；
    
- 这时即使加上 `overflow: auto`，滚动条也不会出现，因为子项本身“被撑大”了。
    

通过设置 `height: 0`，告诉浏览器：

- “我本身没有固有高度，内容高度忽略”；
    
- 这样 `flex: 1` 的计算就只基于父容器的剩余空间，强制限制了子项的高度；
    
- 最终 `overflow: auto` 生效，滚动条出现。
    

---

### 2. 布局层级解析

- **外层容器**
    
    - `display: flex; flex-direction: column; flex: 1;`
        
    - 自身填满父容器，按列方向布局。
        
- **标题部分**
    
    - 固定高度的非滚动区域。
        
- **列表容器（滚动区域）**
    
    - 设置 `flex: 1`，占据剩余空间；
        
    - 设置 `overflow-y: auto`，允许超出时滚动；
        
    - 内部再包一层 `height: 0` 的 div，使得内容不会撑开容器。
        

---

### 3. 效果图（逻辑）

```
+------------------------------------------+
| 管理按钮（可选）                         |
+------------------------------------------+
| 最近使用                                 |
+------------------------------------------+
|                                          |
|   可滚动列表区域（flex:1, overflow:auto） |
|   ┌───────────────────────────────────┐  |
|   │ height:0 的内部容器                │  |
|   │ ──> 内容撑开后出现滚动条           │  |
|   └───────────────────────────────────┘  |
|                                          |
+------------------------------------------+
```

---

## 常见问题与注意事项

1. **能不能直接写 `flex: 1; overflow: auto;`？**
    
    - 在多数场景下不行，因为子项高度会被内容撑开，`overflow` 不触发。
        
2. **为什么不用 `min-height: 0`？**
    
    - `min-height: 0` 是另一种解决办法，适合父容器用 `flex` 布局时，允许子项收缩高度。
        
    - 在某些浏览器（尤其是旧版本）中，`min-height: 0` 兼容性略差，而 `height: 0` 更“保险”。
        
3. **适用场景**
    
    - Flex 布局下的自适应滚动区域；
        
    - 不想写死高度（例如响应式布局、嵌套布局场景）。
        

---

## 总结

通过 **`height: 0` + `flex: 1` + `overflow: auto`** 的组合，可以在 **不固定高度的 Flex 布局中**，优雅地实现 **自适应高度的可滚动列表**。  
这一技巧尤其适用于 **布局容器嵌套、响应式页面、侧边栏面板** 等复杂场景。