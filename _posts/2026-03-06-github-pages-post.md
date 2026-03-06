---
layout: post
title: "如何在 GitHub Pages 上发布文章"
tags: [GitHub Pages, Jekyll, 博客]
---

最近我开始搭建自己的技术博客。

这篇文章记录一下，如何把文章发布到 GitHub Pages。

<!-- more -->

## 1. 新建文章文件

文章要放在 `_posts` 目录下，文件名格式是：

`YYYY-MM-DD-title.md`

## 2. 写 front matter

文章开头要写：

```yaml
---
layout: post
title: "标题"
tags: [标签1, 标签2]
---
```
## 3. 提交到 GitHub

提交后，GitHub Pages 会自动重新构建网站。
