---
author: Axton
pubDatetime: 2025-07-08T15:22:00Z
modDatetime: 2025-07-08T06:25:46.734Z
title: GitHub提交代码无contributions记录解决办法
featured: false
draft: false
tags:
  - GitHub
description:
  问题：最近在使用GigHub时，发现提交的记录并没有统计在GitHub首页的Contributions Graph里（即贡献图上没有绿块）。
---

## 前言

> 问题：最近在使用GigHub时，发现提交的记录并没有统计在GitHub首页的Contributions Graph里（即贡献图上没有绿块）。

> 原因：经过查资料发现，是因为提交时填写的邮箱与GitHub账号里的邮箱不一致导致，而GitHub是以邮箱关联GitHub账号的。
> 解决：在GitHub中添加提交代码的邮箱之后，再次提交，贡献图里统计到了数据！
> 添加方法如下：点击右上角头像图标 -> Settings -> Emails

![GitHub设置截图](@/assets/images/Snipaste_2025-07-08_09-12-02.png)

## 解决

###  设置用户名 （GitHub用户名一致）

```bash
git config --global user.name “username”
```

### 设置邮箱 (没有双引号，GitHub邮箱一致)

```bash
git config --global user.email useremail@outlook.com
```

