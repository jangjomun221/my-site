---
title: "Docker 多阶段构建：从 1.2GB 到 80MB 的镜像瘦身之路"
description: "记录一次 Node.js 项目 Docker 镜像优化过程，通过多阶段构建将镜像体积缩减 93%，同时提升构建缓存命中率。"
pubDate: 2026-05-22
---

最近接手一个 Node.js 微服务，发现它的 Docker 镜像有 1.2GB。部署一次要传输半天，CI/CD 流水线也被拖慢。花了一个下午优化，最终压到了 80MB。记录一下过程。

## 问题分析

原始 Dockerfile 大致长这样：

```dockerfile
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

问题很明显：
- 基础镜像 `node:20` 包含完整的 Debian 系统，约 900MB
- `npm install` 安装了 devDependencies
- 源码、node_modules 全部打进最终镜像

## 第一步：使用 Alpine 基础镜像

```dockerfile
FROM node:20-alpine
```

仅这一步就从 900MB 降到 ~180MB。Alpine 基于 musl libc，体积极小。

## 第二步：多阶段构建

核心思路是把「构建环境」和「运行环境」分离：

```dockerfile
# 构建阶段
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 生产阶段
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

关键点：
- `npm ci` 比 `npm install` 更快且确定性更强
- 生产阶段只安装 `dependencies`，不含 `devDependencies`
- 只复制编译产物 `dist/`，源码不进最终镜像

## 第三步：进一步压缩 —— distroless

如果项目没有依赖 native 模块，可以用 Google 的 distroless 镜像：

```dockerfile
FROM gcr.io/distroless/nodejs20-debian12 AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["dist/main.js"]
```

Distroless 镜像不包含 shell、包管理器等工具，安全性也更好。

## 第四步：优化构建缓存

Docker 的层缓存机制意味着指令顺序很重要：

```dockerfile
# 先复制 package 文件
COPY package*.json ./
RUN npm ci

# 再复制源码
COPY . .
RUN npm run build
```

这样只要 `package.json` 没变，`npm ci` 那层就会命中缓存。如果反过来先 `COPY . .`，任何源码改动都会让依赖安装重跑。

## 最终效果对比

| 版本 | 镜像大小 | 构建时间（冷启动） |
|------|----------|-------------------|
| 原始 | 1.2 GB | 4m 30s |
| Alpine | 310 MB | 3m 20s |
| 多阶段 | 120 MB | 2m 10s |
| distroless | 80 MB | 2m 15s |

## 注意事项

- Alpine 使用 musl libc，某些依赖 glibc 的 native 模块可能不兼容
- distroless 镜像无法 `exec` 进容器调试，排查问题时需要临时切回带 shell 的镜像
- 多阶段构建的 `COPY --from` 不会继承文件权限，注意需要的话加 `--chmod`

镜像瘦身不只是节省磁盘。更小的镜像意味着更快的拉取、更快的滚动更新、更小的攻击面。值得投入时间优化。