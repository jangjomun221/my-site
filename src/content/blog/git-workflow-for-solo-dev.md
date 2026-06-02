---
title: "独立开发者的 Git 工作流：简单但不随意"
description: "分享一套适合个人项目和小团队的 Git 工作流，在保持简洁的同时不丧失版本管理的核心价值。"
pubDate: 2026-06-01
---

很多独立开发者的 Git 使用方式就是 `add -A && commit -m "update" && push`。能跑就行，但一旦需要回滚或排查问题，就会发现历史记录毫无参考价值。这里分享一套我用了两年的工作流，简单但有效。

## 核心原则

1. **main 分支永远可部署** —— 不在 main 上直接开发
2. **一个功能一个分支** —— 即使只有你一个人
3. **提交信息有意义** —— 未来的你会感谢现在的你

## 分支策略

```
main          ← 稳定、可部署
├── feat/xxx  ← 新功能
├── fix/xxx   ← 修复
└── exp/xxx   ← 实验性尝试
```

分支命名用前缀区分类型，`xxx` 用简短的英文描述。比如 `feat/dark-mode`、`fix/login-redirect`、`exp/new-layout`。

`exp/` 分支允许随意，可以 force push，可以不合并就删除。其他分支遵循正常流程。

## 提交规范

不需要 Angular 那套完整的 Conventional Commits，但至少做到：

```
feat: 添加暗色模式切换
fix: 修复登录后重定向丢失 query 参数
refactor: 抽取请求拦截器为独立模块
chore: 升级 vite 到 6.x
```

前缀让 `git log --oneline` 一目了然。写中文完全没问题，关键是描述「做了什么」而非「改了哪个文件」。

## 我的日常流程

```bash
# 开始新功能
git checkout -b feat/search

# 开发过程中，频繁小提交
git add -p                    # 按 hunk 暂存，不用 -A
git commit -m "feat: 搜索框 UI 骨架"
git commit -m "feat: 接入搜索 API"
git commit -m "feat: 添加搜索结果高亮"

# 功能完成，合并前整理
git rebase -i HEAD~3          # 如果需要合并琐碎提交

# 合并到 main
git checkout main
git merge feat/search --no-ff # --no-ff 保留分支痕迹
git push
git branch -d feat/search
```

## 关于 `git add -p`

这是我认为最被低估的 Git 命令。它会逐个展示改动的 hunk，让你选择哪些进入暂存区。好处：

- 避免把调试代码、`console.log` 意外提交
- 一次编码可以拆成多个语义清晰的提交
- 强迫你 review 自己的代码

## 关于 rebase vs merge

个人项目我倾向 rebase 保持线性历史：

```bash
# 在 feature 分支上拉取 main 的最新改动
git rebase main
```

但合并到 main 时用 `--no-ff`，这样 `git log --graph` 能看到每个功能分支的起止。

## .gitignore 的维护

项目初期就建好 `.gitignore`，避免后续用 `git rm --cached` 清理：

```
node_modules/
dist/
.env
.env.local
*.log
.DS_Store
```

## 有用的别名

在 `~/.gitconfig` 中配置：

```ini
[alias]
  st = status -sb
  lg = log --oneline --graph --all -20
  co = checkout
  cm = commit -m
  ap = add -p
  last = log -1 --stat
```

`git lg` 尤其实用，一行展示最近 20 条提交的图形化历史。

## 总结

这套工作流的核心是：用最小的规则获得最大的收益。不需要 PR review（你就一个人），不需要 CI 必须过才能合并（但有 CI 更好），只要做到分支隔离和有意义的提交信息，日后维护就会轻松很多。