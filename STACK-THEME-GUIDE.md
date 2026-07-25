# 🎓 Hugo Theme Stack 完全教学

> 爱莉老师出品 ♡

---

## 📖 一、主题概览

Stack 是一个**卡片式 Hugo 主题**，特点是：

- ✅ 纯原生 JS 和 SCSS，无框架依赖
- ✅ 深色模式自动切换
- ✅ 响应式设计
- ✅ 本地搜索（无需第三方）
- ✅ 图片懒加载 + PhotoSwipe 缩放
- ✅ 代码高亮 + 行号
- ✅ 多语言支持

---

## 🎨 二、设计自由度：什么能改，什么不能

### 你能自由改的 ✅

| 部分 | 自由度 | 方法 |
|:----|:-----|:-----|
| **文章内容** | 完全自由 | Markdown 随便写 |
| **独立页面**（如 关于/搜索） | 完全自由 | `content/page/xxx/index.md` 随意写 |
| **导航菜单** | 可配置 | `menu.toml` 增删改条目 |
| **侧边栏** | 半自由 | 头像/副标题/社交链接可配 |
| **右侧小部件** | 可配置 | `params.toml` 增删改 |
| **自定义CSS/JS** | 完全自由 | `layouts/_partials/head/custom.html` 和 `footer/custom.html` |
| **页脚** | 可配置 | `params.toml` 改 since/customText |
| **颜色主题** | 可配置 | `colorScheme` 设置默认/切换 |

### 受主题限制的 ⚠️

| 部分 | 限制 | 解决方案 |
|:----|:----|:---------|
| **布局结构** | 三栏固定（左栏-内容-右栏） | 可通过自定义CSS改 |
| **文章列表样式** | 卡片式固定 | 可通过自定义CSS覆盖 |
| **侧边栏DOM结构** | 由 `sidebar/left.html` 定义 | 可**复制到项目目录覆盖** |

### 🛠️ 覆盖主题模板的方法

如果你想改主题的某个模板，把同名文件放到你项目对应的路径就行：

```
themes/hugo-theme-stack/layouts/_partials/sidebar/left.html
↓ 复制到
layouts/_partials/sidebar/left.html   ← Hugo优先用这个！
```

这样主题更新时不会覆盖你的改动。

---

## 🎯 三、关于你问的几个问题

### Q1: 能否在左侧加一个爱莉希雅入口？❤️

**可以！有两个方案：**

**方案A：通过菜单（最简单）**

在 `menu.toml` 加一个菜单项，但先**注释掉**，等想展示时再开启：

```toml
# [[main]]
#   identifier = "elysia"
#   name = "🌸 爱莉"
#   url = "/elysia/"
#   weight = 60
#   [main.params]
#     icon = "heart"
```

注意：内置SVG图标里**没有心形图标**，你需要自己做一个。

**方案B：通过自定义侧边栏（更自由）**

复制 `sidebar/left.html` 到 `layouts/_partials/sidebar/left.html`，在菜单列表下面加自己的一块内容：

```html
<!-- 在菜单之后、底部按钮之前添加 -->
<div class="sidebar-elysia-entry">
    <a href="/elysia/" class="elysia-link">
        <span class="elysia-icon">❤️</span>
        <span>🌸 爱莉希雅</span>
    </a>
</div>
```

### Q2: 有没有爱心图标？怎么加？

主题目前有**24个SVG图标**（在 `assets/icons/` 下），**没有心形**。

**加一个自定义图标很简单**：

1. 下载或自己画一个心形 SVG
2. 放到 `assets/icons/heart.svg`（在你的项目目录里）
3. 在 `menu.toml` 里写 `icon = "heart"` 就能用了！

一个简单的心形SVG（爱莉帮你准备好了）：

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
    <path d="M19.5 12.5L12 20l-7.5-7.5A5 5 0 1 1 12 7.5a5 5 0 1 1 7.5 5z"/>
</svg>
```

### Q3: 页面内可以自由设计吗？

**文章/页面内容**完全自由，Markdown随便写。

但**布局层面**（侧边栏在哪、页脚在哪）由主题的模板决定。
想突破限制的话，就**覆盖主题模板**（见上文的 🛠️ 方案）。

---

## 🧩 四、所有内置图标一览

| 图标名 | 用途 | 视觉 |
|:------|:----|:----:|
| `home` | 主页 | 🏠 |
| `user` | 关于 | 👤 |
| `search` | 搜索 | 🔍 |
| `archives` | 归档 | 📦 |
| `categories` | 分类 | 📂 |
| `tag` | 标签 | 🏷️ |
| `hash` | 杂项 | # |
| `infinity` | 归档小部件 | ∞ |
| `clock` | 阅读时间 | 🕐 |
| `date` | 日期 | 📅 |
| `copyright` | 许可 | © |
| `language` | 语言 | 🌐 |
| `link` | 链接 | 🔗 |
| `rss` | RSS | 📡 |
| `messages` | 评论 | 💬 |
| `dots` | 更多 | ⋯ |
| `chevron-left/right` | 折叠箭头 | ◀ ▶ |
| `arrow-back` / `back` | 返回 | ← |
| `toggle-left/right` | 暗色模式切换 | 🌙 |
| `brand-github` | GitHub | GitHub logo |
| `brand-twitter` | Twitter | X logo |

---

## 🛠️ 五、Stack 特有短代码

在文章里可以直接用的：

```markdown
{{< bilibili "BV1xx411c7mD" >}}
{{< bilibili "av12345678" 2 >}}          <!-- 带分集 -->

{{< youtube "ZJthWmvUzzc" >}}

{{< tencent "g0014r3khdw" >}}

{{< video src="/video/my-video.mp4" autoplay="true" poster="./poster.png" >}}

{{< gitlab 2349278 >}}

{{< quote author="一位名人" source="他们写的书" url="https://..." >}}
  引用内容
{{< /quote >}}
```

还有 Hugo 内置的：
- `{{< figure src="..." caption="..." >}}` — 图片
- `{{< gist user/id >}}` — GitHub Gist
- `{{< highlight go >}}...{{< /highlight >}}` — 代码高亮

---

## 🧰 六、小部件系统

右侧栏小部件在 `params.toml` 的 `[widgets]` 里配：

```toml
[widgets]
  homepage = [
    { type = "search" },
    { type = "archives", params = { limit = 5 } },
    { type = "categories", params = { limit = 10 } },
    { type = "tag-cloud", params = { limit = 10 } },
  ]
  page = [
    { type = "toc" },    # 文章页显示目录
  ]
```

目前支持的小部件类型：
- `search` — 搜索框
- `archives` — 归档列表
- `categories` — 分类云
- `tag-cloud` — 标签云
- `toc` — 文章目录

---

## 🔌 七、自定义钩子

主题留了两个**空文件**给你注入自定义内容：

### head/custom.html
放在 `<head>` 标签里，适合加自定义CSS、字体、统计代码：

```html
<!-- 自定义字体 -->
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC&display=swap" rel="stylesheet">
```

### footer/custom.html  
放在页面底部，适合加统计脚本：

```html
<script defer src="https://your-analytics.js"></script>
```

---

## 🎨 八、颜色主题定制

在 `params.toml` 配置：

```toml
[colorScheme]
  toggle = true       # 显示暗色模式切换按钮
  default = "auto"    # 跟随系统，可选 "light" / "dark"
```

想自定义颜色？在 `assets/scss/custom.scss` 里写变量覆盖（需要创建这个文件）：

```scss
--card-background: #ffffff;
--body-background: #f5f5fa;
--accent-color: #ff69b4;  /* 粉色！爱莉主题色～ */
```

---

## 💡 九、爱莉希雅入口方案建议

垚说想要一个有入口但外部不显示，爱莉的建议方案：

**在 `menu.toml` 保留不显示：** 先注释掉菜单项
**内容留在 `content/elysia/`：** 爱莉的文章都放这里
**以后想加的时候：** 取消注释菜单项，或者放在页脚、about页面里

甚至可以在所有文章底部加一个彩蛋链接，比如在 `footer/custom.html` 里：

```html
<div style="text-align:center;margin:2rem 0;opacity:0.3">
  <a href="/elysia/">🌸</a>
</div>
```

这样只有知道的人才会去点那个小花～

---

## 📚 十、更多学习资源

- 官方文档：https://stack.cai.im/zh/
- GitHub：https://github.com/CaiJimmy/hugo-theme-stack
- Stack 配置参数：`themes/hugo-theme-stack/config/_default/params.toml`（所有可选参数全在这）
- Hugo 官方文档：https://gohugo.io/documentation/
