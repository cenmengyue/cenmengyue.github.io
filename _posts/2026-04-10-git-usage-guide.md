---
layout: post
title: Git 使用指南：写作、提交与协作的最小闭环
date: 2026-04-10
author: CIG
categories:
- tooling
tags:
- git
- github
- workflow
- notes
---

> 这是一份给自己看的 Git 日常使用指南，重点放在“少出错、好回滚、好协作”。

## 1. 先记住最常用的闭环

日常工作里，Git 最常见的流程其实很简单：

```bash
git status
git diff
git add <file>
git commit -m "message"
git push
```

含义分别是：
- `git status`：看当前有哪些文件被修改了。
- `git diff`：看具体改了什么。
- `git add`：把准备提交的文件放进暂存区。
- `git commit`：把暂存区内容保存成一次提交。
- `git push`：把本地提交推到远端仓库。

## 2. 初始化和克隆

如果是新仓库，可以先初始化：

```bash
git init
```

如果是已有仓库，通常直接克隆：

```bash
git clone https://github.com/owner/repo.git
```

克隆后进入目录，再查看远程信息：

```bash
git remote -v
git branch
```

## 3. 提交前先看清楚改动

提交前我建议固定做两件事：

```bash
git status -sb
git diff
```

如果工作区里有很多改动，优先只暂存自己要提交的文件：

```bash
git add about.md
git add _posts/2026-04-10-git-usage-guide.md
```

不要在不确定范围时直接用 `git add -A`，否则很容易把不该发的内容一起提交。

## 4. 提交信息怎么写

提交信息要短、明确、可回看。比较实用的格式是：

```bash
git commit -m "Add git usage guide"
```

如果是修复类改动，可以写成：

```bash
git commit -m "Fix post front matter"
```

原则很简单：
- 一次提交只做一类事情。
- 提交信息要能让人一眼知道改了什么。
- 不要把多个无关改动揉进同一个提交。

## 5. 分支的基本用法

分支是 Git 里最重要的协作工具之一。常用命令如下：

```bash
git switch -c feature/git-guide
git switch master
git branch
```

推荐习惯是：
- 新功能、新文章、新实验，尽量开一个独立分支。
- 主分支只放已经整理好的内容。
- 合并前先确认 diff 清楚、提交干净。

## 6. 拉取远端更新

如果远端已经有人更新了内容，先同步再继续改：

```bash
git fetch origin
git pull
```

我更偏向先 `fetch` 再看差异，再决定怎么合并。这样更容易判断远端到底改了什么。

## 7. 回滚和撤销

Git 强大的一点是可撤销，但要分清场景。

如果只是想取消工作区里的本地修改：

```bash
git restore <file>
```

如果已经 `add` 进暂存区，但还没提交：

```bash
git restore --staged <file>
```

如果已经提交了，但想撤销这次提交带来的影响，优先用：

```bash
git revert <commit>
```

`revert` 会生成一个新的提交来抵消旧提交，适合已经推到远端的情况。

## 8. 一个安全的日常流程

我现在更推荐下面这个节奏：

```bash
git status -sb
git diff
git add <file>
git commit -m "..."
git push
```

如果是多人协作，再加上：

```bash
git fetch origin
git log --oneline --graph --decorate --all
```

这样可以更清楚地看到历史分支和提交关系。

## 9. 最后给自己的检查清单

提交前快速确认这几项：

- 只提交了本次想发布的文件。
- 提交信息能看懂。
- 没有误删重要内容。
- 没有把临时文件、缓存文件、密钥文件推上去。
- 如果是协作仓库，先同步远端再推送。

Git 的核心不是背命令，而是养成稳定的流程：先看，再改；先确认，再提交；先验证，再推送。
