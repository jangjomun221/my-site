# My Site

基于 Astro 6 构建的个人网站，包含技术博客、感悟随笔、作品集和履历页面。

## 技术栈

- [Astro](https://astro.build/) 6.x — 静态站点生成
- MDX — 支持组件的 Markdown 内容
- Sitemap — 自动生成站点地图
- Sharp — 图片优化处理

## 项目结构

```
src/
├── content/
│   ├── blog/              # 技术博客 (Markdown)
│   ├── thoughts/          # 感悟随笔 (Markdown)
│   └── content.config.ts  # Content Collections 配置
├── layouts/
│   └── Base.astro         # 全局布局
├── pages/
│   ├── blog/
│   │   ├── index.astro    # 博客列表
│   │   └── [...slug].astro # 博客详情动态路由
│   ├── thoughts/
│   │   ├── index.astro    # 感悟列表
│   │   └── [...slug].astro # 感悟详情动态路由
│   ├── works/
│   │   └── index.astro    # 作品集
│   ├── 404.astro          # 自定义 404 页面
│   ├── resume.astro       # 履历
│   └── index.astro        # 首页
public/
├── images/                # 静态图片资源
│   ├── blog/
│   ├── thoughts/
│   └── works/
├── favicon.ico
└── favicon.svg
```

## 开发

```bash
npm install        # 安装依赖
npm run dev        # 启动开发服务器 localhost:4321
npm run build      # 构建生产版本
npm run preview    # 本地预览构建结果
```

要求 Node.js >= 22.12.0。

## 添加内容

在 `src/content/blog/` 或 `src/content/thoughts/` 下新建 `.md` 文件，frontmatter 格式：

```yaml
---
title: "文章标题"
description: "简短描述"
pubDate: 2026-06-01
---
```

对应的图片放入 `public/images/blog/` 或 `public/images/thoughts/` 目录。
