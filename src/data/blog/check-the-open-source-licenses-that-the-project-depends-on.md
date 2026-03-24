---
author: Axton
pubDatetime: 2026-03-05T15:22:00Z
modDatetime: 2026-03-05T15:22:00Z
title: 检查项目依赖的开源许可证
featured: false
draft: false
tags:
  - docs
description:
  下面这些命令是用来**检查和导出项目所有依赖包的开源许可证信息**的。
---

下面这些命令是用来**检查和导出项目所有依赖包的开源许可证信息**的。

## `license-checker` 是什么？

它是一个 Node.js 工具，能自动扫描你项目的 `node_modules`，列出每个依赖包使用的开源许可证（MIT、Apache-2.0、GPL 等）。

---

## 逐条解释

**1. 全局安装**

```bash
npm install -g license-checker
```

把工具装到全局，之后在任意项目里都能直接用。

**2. 生成许可证摘要**

```bash
license-checker --summary
```

在终端打印一个汇总，告诉你项目里各种许可证类型分别有多少个包，比如：

```
MIT: 312
ISC: 45
Apache-2.0: 12
BSD-3-Clause: 8
```

**3. 导出完整 CSV 报告**

```bash
pnpm dlx license-checker --csv --out license.csv
```

不需要全局安装，用 `pnpm dlx` 临时运行，把**每个依赖包的详细许可证信息**导出到 `license.csv` 文件，包含包名、版本、许可证类型、仓库地址等。

---

## 这有什么实际用途？

|场景|说明|
|---|---|
|**法务合规**|商业项目上线前，需要确认没有使用 GPL 等"传染性"许可证|
|**开源审计**|甲方或客户要求提供依赖的许可证清单|
|**风险排查**|找出可能有版权风险的依赖|
|**自动化 CI**|配合 `--failOn "GPL"` 参数，一旦引入禁止的许可证就让构建失败|

简单说：**你在用别人的代码，这个工具帮你搞清楚你"欠"别人什么许可条款。**