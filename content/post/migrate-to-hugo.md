---
title: "Hugo + GitHub Pages 搭建个人博客"
description: "从 Hexo 迁移到 Hugo 的完整记录"
date: 2026-07-25T10:00:00+08:00
draft: false
tags:
  - Hugo
  - GitHub Pages
  - 折腾
categories:
  - 技术
---

## 为什么换？ 

之前用 Hexo + Butterfly 主题折腾了挺久，花里胡哨的其实也没写几篇文章（笑）。
这次决定换个清爽的，顺便趁这个机会统一一下线上 ID。

### 选型

- **框架**: Hugo → 快，单二进制，不用 Node
- **主题**: Stack → 卡片风格，简约
- **部署**: GitHub Actions → push 即上线
- **域名**: horikitax.github.io → User Site

### 迁移过程

1. 旧博客搬到 `./课业/Elysia_blog` 留着纪念
2. `hugo new site` 初始化
3. 装主题、配参数
4. 写 .github/workflows/deploy.yml
5. push → 上线 ✨

### 感受

Hugo 确实快，本地 `hugo server` 基本秒开。
Stack 主题的卡片布局很舒服，不需要自己折腾 UI 了。

以后就在这里认真写技术笔记啦～
