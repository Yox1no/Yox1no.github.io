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

## 选型

- **框架**: Hugo → 快，单二进制，不用 Node
- **主题**: Stack → 卡片风格，简约
- **部署**: GitHub Actions → push 即上线
- **域名**: horikitax.github.io → User Site

## 迁移过程

### 第一步：备份旧博客

先把旧 Hexo 博客整个目录挪走，保留在本地纪念：

```bash
mv ~/Code/Blog ~/Code/课业/Elysia_blog
```

### 第二步：初始化 Hugo

```bash
hugo new site blog --force
cd blog
git init
```

### 第三步：安装 Stack 主题

```bash
git submodule add https://github.com/CaiJimmy/hugo-theme-stack themes/hugo-theme-stack
```

### 第四步：配置 GitHub Actions

创建 `.github/workflows/deploy.yml`，配置自动构建和部署。

### 第五步：配置参数

在 `config/_default/` 下配置 params.toml、menu.toml 等。

## 感受

Hugo 确实快，本地 `hugo server` 基本秒开。
Stack 主题的卡片布局很舒服，不需要自己折腾 UI 了。

以后就在这里认真写技术笔记啦～
