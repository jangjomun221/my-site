---
title: "CSS Container Queries 实战：告别媒体查询的组件级响应式"
description: "介绍 CSS Container Queries 的核心概念和使用方式，通过实际案例展示如何实现真正的组件级响应式设计。"
pubDate: 2026-05-08
---

媒体查询（Media Queries）一直是响应式设计的基石，但它有一个根本性缺陷：只能基于视口宽度做判断。一个组件放在侧边栏和主内容区，即使可用空间完全不同，媒体查询给出的结果却一样。Container Queries 解决了这个问题。

## 核心概念

Container Queries 让组件根据其**父容器**的尺寸来调整样式，而非视口：

```css
/* 声明一个容器 */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

/* 当容器宽度小于 400px 时 */
@container card (max-width: 400px) {
  .card {
    flex-direction: column;
  }
  .card img {
    width: 100%;
  }
}

/* 当容器宽度大于 600px 时 */
@container card (min-width: 600px) {
  .card {
    grid-template-columns: 1fr 2fr;
  }
}
```

## container-type 的选项

- `inline-size` —— 只对内联方向（通常是宽度）建立包含上下文，最常用
- `size` —— 同时对宽度和高度建立包含上下文
- `normal` —— 默认值，不建立尺寸包含上下文

绝大多数场景用 `inline-size` 就够了。`size` 会产生额外的布局约束，慎用。

## 实际案例：文章卡片组件

一个卡片组件需要适应不同的放置位置：

```html
<div class="grid-main">
  <div class="card-container">
    <article class="card">...</article>
  </div>
</div>

<aside class="sidebar">
  <div class="card-container">
    <article class="card">...</article>
  </div>
</aside>
```

```css
.card-container {
  container-type: inline-size;
}

.card {
  display: grid;
  gap: 1rem;
  padding: 1.5rem;
}

/* 窄容器：纵向堆叠 */
@container (max-width: 300px) {
  .card {
    grid-template-columns: 1fr;
  }
  .card__image {
    aspect-ratio: 16/9;
    order: -1;
  }
  .card__title {
    font-size: 1rem;
  }
}

/* 宽容器：横向排列 */
@container (min-width: 500px) {
  .card {
    grid-template-columns: 200px 1fr;
    align-items: start;
  }
  .card__title {
    font-size: 1.25rem;
  }
}
```

同一个 `.card` 组件，在侧边栏自动切换为纵向布局，在主内容区自动展开为横向布局。无需额外的 class 或 JavaScript。

## Container Query Units

除了尺寸查询，还有一组相对于容器的单位：

| 单位 | 含义 |
|------|------|
| `cqw` | 容器宽度的 1% |
| `cqh` | 容器高度的 1% |
| `cqi` | 容器内联方向尺寸的 1% |
| `cqb` | 容器块方向尺寸的 1% |
| `cqmin` | cqi 和 cqb 中的较小值 |
| `cqmax` | cqi 和 cqb 中的较大值 |

```css
.card__title {
  font-size: clamp(1rem, 3cqi, 1.5rem);
}
```

这样标题字号会跟随容器尺寸平滑缩放，同时被 `clamp` 限制在合理范围内。

## 与媒体查询的配合

Container Queries 不是取代媒体查询，而是补充。典型的分工是：

- **媒体查询**：控制页面整体布局（几栏、导航形态）
- **Container Queries**：控制组件内部的适配

```css
/* 媒体查询控制布局 */
@media (min-width: 768px) {
  .page {
    display: grid;
    grid-template-columns: 1fr 300px;
  }
}

/* Container Queries 控制组件 */
.card-container { container-type: inline-size; }

@container (min-width: 400px) {
  .card { /* 宽布局 */ }
}
```

## 浏览器支持

截至 2026 年，所有主流浏览器均已支持 Container Queries（Chrome 105+、Firefox 110+、Safari 16+）。可以放心在生产环境使用，不再需要 polyfill。

## 总结

Container Queries 让「组件真正自包含」成为现实。设计系统中的组件不再需要关心自己被放在哪里，只需要根据可用空间自适应。这是 CSS 布局能力的一次重要进化。
