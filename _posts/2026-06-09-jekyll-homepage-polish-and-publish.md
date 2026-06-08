---
layout: post
title: "一次 Jekyll 博客首页改造与发布记录"
date: 2026-06-09
author: CIG
categories:
- tooling
tags:
- Jekyll
- GitHub Pages
- WSL
- Ruby
- Codex
- blog
---

> 记录今天对个人博客首页做的一次改造：在保留 Jekyll 博客结构的前提下，美化首页、补齐动态效果、搭好本地构建环境，并推送到 GitHub Pages 发布。

## 1. 起因

今天的目标很直接：把个人主页的外观做得更现代一些，最好加一点动态效果。

这个博客本身是 GitHub Pages + Jekyll 结构，文章都放在 `_posts` 目录下，首页通过 `index.html` 和分页器读取文章列表。这个前提很重要，因为个人博客不是一个单独的静态落地页，首页必须继续服务于文章列表、标签、归档和搜索。

最开始我差点把首页做成一份独立 HTML。这个方向视觉上可以很快出效果，但它会绕开 Jekyll 的文章生成机制。后面马上改回了 Jekyll 模板，让首页继续使用：

```liquid
{% raw %}{% for post in paginator.posts %}{% endraw %}
```

这样新的首页仍然能自动读取 `_posts` 里的文章，不需要手动维护文章卡片。

## 2. 首页做了什么

这次主要改了两个文件：

- `index.html`
- `style.scss`

首页结构上新增了几个部分：

- 顶部 hero 区：展示博客名称、方向和简介。
- 搜索区：继续使用原来的 `simple-jekyll-search`。
- 主题筛选按钮：可以按 AI Agent、多模态、GitHub Pages、Git、vLLM 等方向筛选当前页文章。
- 文章卡片：把原来的线性列表改成更清晰的卡片布局。
- 阅读动效：滚动入场动画、卡片 hover、顶部阅读进度条。
- 访问统计：保留不蒜子的 PV/UV 显示。

样式上没有引入额外前端框架，还是沿用 Jekyll 的 Sass 编译链。这样 GitHub Pages 构建时只需要处理原有的 `style.scss`，不需要 Node、Webpack 或 Vite。

## 3. 保留 Jekyll 结构

这次最关键的点是：美化不能破坏博客系统。

所以首页仍然保留了这些 Jekyll 能力：

- `layout: default`
- `paginator.posts`
- `post.title`
- `post.url`
- `post.tags`
- `post.excerpt`
- 上一页 / 下一页分页
- `search.json` 搜索数据

文章链接仍然由 Jekyll 的 permalink 规则生成，比如：

```text
/git-usage-guide/
/seg-zero/
/agent-mvp-qwen3-5-35b-vllm/
```

这意味着以后只要继续往 `_posts` 里写 Markdown，首页就会自动更新。

## 4. 本地构建环境

一开始 Windows 环境里没有 Ruby、Jekyll、Bundler，所以不能直接验证构建。后面改用 WSL 里的 Ubuntu 24.04 来跑 Jekyll。

安装了这些工具：

- Ruby
- Jekyll
- Bundler
- `jekyll-sitemap`
- `jekyll-feed`
- `jekyll-paginate`
- `webrick`

因为安装 RubyGems 包比较慢，后面把 RubyGems 源切到了清华源：

```bash
gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
```

同时，WSL 里也临时走了 Windows 本地代理。Windows 侧代理端口是 `7890`，WSL 里通过网关访问：

```bash
http://172.31.160.1:7890
```

这个代理只用于安装和联网验证，没有写进仓库。

## 5. 构建和预览

环境准备好之后，用 Jekyll 做了本地构建：

```bash
jekyll build
```

构建成功。输出里有 Sass 的弃用警告，主要来自旧主题里的 `@import` 和 `map-get` 写法。这些是历史 Sass 语法提醒，不影响当前 GitHub Pages 构建和页面生成。

然后启动本地预览：

```bash
jekyll serve --host 0.0.0.0 --port 4000
```

本地访问地址：

```text
http://localhost:4000/
```

验证结果：

- 首页返回 200。
- `style.css` 正常加载。
- 首页里能找到新版 `home-hero`、`home-post-card`、`最近文章` 等结构。
- `_site` 构建产物没有进入 Git 提交，因为它已经在 `.gitignore` 里。

## 6. 发布

最后把改动提交并推送到了 GitHub：

```bash
git add index.html style.scss
git commit -m "Polish Jekyll blog homepage"
git push origin master
```

对应提交：

```text
fee091f Polish Jekyll blog homepage
```

推送后访问线上首页：

```text
https://cenmengyue.github.io/
```

线上也已经能匹配到新版首页结构，说明 GitHub Pages 已经完成构建和发布。

## 7. 小结

今天这次改造的核心不是单纯把页面做得好看一点，而是在不破坏 Jekyll 写作流程的基础上，把博客首页整理得更像一个长期维护的个人技术主页。

现在这个博客的发布流程更清楚了：

1. 在 `_posts` 写 Markdown。
2. 用 Jekyll 本地构建验证。
3. 确认首页、标签、归档和搜索正常。
4. 提交并推送到 `master`。
5. 等 GitHub Pages 自动发布。

后续如果继续优化，可以考虑再做三件事：

- 给文章页也做一点更现代的阅读样式。
- 清理旧 Sass 的弃用警告。
- 给首页筛选功能加上更多标签映射，让它不只筛当前页文章。
