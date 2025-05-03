---
author: Axton
pubDatetime: 2023-04-21T15:22:00Z
modDatetime: 2023-04-21T15:22:00Z
title: Element Dialog水平垂直居中样式
featured: false
draft: false
tags:
  - Element UI
  - Dialog
description:
   Element UI 是目前最火的前端 Vue.js UI 组件库，但是 Dialog 默认样式并不是水平垂直居中，这就很让人很尴尬。不过对于有水平垂直居中的需求来说，我们是可以自己进行调整的。最常用的方法也试过了，最终得到以下方法是最佳的（我个人感觉），接下来和大家分享一下。
---

## 前言

> Element UI 是目前最火的前端 Vue.js UI 组件库，但是 Dialog 默认样式并不是水平垂直居中，这就很让人很尴尬。不过对于有水平垂直居中的需求来说，我们是可以自己进行调整的。最常用的方法也试过了，最终得到以下方法是最佳的（我个人感觉），接下来和大家分享一下。

### 一、默认样式截图

![默认样式截图](@/assets/images/20210127172724470.png)

### 二、调整后样式截图

![调整后样式截图](@/assets/images/2021012717274412.png)

### 二、调整后样式截图

App.vue

```scss
.el-dialog__wrapper {
  text-align: center;
  white-space: nowrap;
  overflow: auto;
  &:after {
    content: "";
    display: inline-block;
    vertical-align: middle;
    height: 100%;
  }
  .el-dialog {
    margin: 30px auto !important;
    display: inline-block;
    vertical-align: middle;
    text-align: left;
    white-space: normal;
  }
}
```

### 关于本文

> 作者：fooooooood
>
> 本文链接：https://blog.csdn.net/Fooooooood/article/details/113257185
