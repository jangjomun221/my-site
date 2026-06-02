# My Site

基于 Astro 构建的个人网站，包含博客、感悟、作品集和履历页面。

## 项目结构

```
src/
├── content/
│   ├── blog/          # 技术博客文章 (Markdown)
│   └── thoughts/      # 感悟随笔 (Markdown)
├── layouts/
│   └── Base.astro     # 全局布局
├── pages/
│   ├── blog/          # 博客列表 + 动态路由
│   ├── thoughts/      # 感悟列表 + 动态路由
│   ├── works/         # 作品集
│   ├── resume.astro   # 履历
│   └── index.astro    # 首页
└── content.config.ts  # Content Collections 配置
```

## 开发

```bash
npm install        # 安装依赖
npm run dev        # 启动开发服务器 localhost:4321
npm run build      # 构建生产版本
npm run preview    # 本地预览构建结果
```

## 添加内容

在 `src/content/blog/` 或 `src/content/thoughts/` 下新建 `.md` 文件即可，frontmatter 格式：

```yaml
---
title: "文章标题"
description: "简短描述"
pubDate: 2026-06-01
---
```
