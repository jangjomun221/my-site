---
title: "Vue 3 Composables 实战：五个提升代码复用性的模式"
description: "深入探讨 Vue 3 组合式 API 中 Composables 的设计模式，包括状态共享、异步请求封装、响应式工具函数等实用技巧。"
pubDate: 2026-05-15
---

在 Vue 3 的组合式 API 中，Composables 是实现逻辑复用的核心机制。相比 Vue 2 的 Mixins，它解决了命名冲突、来源不明确等痛点。本文总结了日常开发中最常用的五种模式。

## 模式一：响应式状态封装

最基础的模式是将一组相关的响应式状态和操作封装在一起：

```typescript
import { ref, computed } from 'vue'

export function useCounter(initial = 0) {
  const count = ref(initial)
  const doubled = computed(() => count.value * 2)

  function increment() { count.value++ }
  function reset() { count.value = initial }

  return { count, doubled, increment, reset }
}
```

关键点在于返回值使用 `ref` 而非 `reactive`，这样解构后仍保持响应性。

## 模式二：异步请求封装

几乎所有项目都需要处理 API 请求的加载、错误、数据三态：

```typescript
import { ref, shallowRef } from 'vue'

export function useFetch<T>(url: string) {
  const data = shallowRef<T | null>(null)
  const error = ref<Error | null>(null)
  const loading = ref(false)

  async function execute() {
    loading.value = true
    error.value = null
    try {
      const res = await fetch(url)
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      data.value = await res.json()
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }

  execute()
  return { data, error, loading, refetch: execute }
}
```

使用 `shallowRef` 存储复杂数据可避免不必要的深层响应式转换。

## 模式三：跨组件状态共享

利用模块级变量实现轻量级的全局状态：

```typescript
import { ref } from 'vue'

const globalCount = ref(0)

export function useSharedCounter() {
  function increment() { globalCount.value++ }
  return { count: globalCount, increment }
}
```

多个组件调用 `useSharedCounter()` 时共享同一个 `globalCount`。适合不需要 Pinia 的简单场景。

## 模式四：副作用自动清理

结合 `onUnmounted` 确保资源释放：

```typescript
import { ref, onUnmounted } from 'vue'

export function useEventListener(
  target: EventTarget,
  event: string,
  handler: EventListener
) {
  target.addEventListener(event, handler)
  onUnmounted(() => target.removeEventListener(event, handler))
}
```

这个模式保证了组件卸载时自动解绑事件监听，避免内存泄漏。

## 模式五：组合 Composables

Composable 可以组合其他 Composable，构建更高层的抽象：

```typescript
import { ref, watch } from 'vue'
import { useFetch } from './useFetch'
import { useDebounce } from './useDebounce'

export function useSearch(baseUrl: string) {
  const query = ref('')
  const debouncedQuery = useDebounce(query, 300)

  const { data, loading } = useFetch(
    computed(() => `${baseUrl}?q=${debouncedQuery.value}`)
  )

  return { query, results: data, loading }
}
```

## 总结

这五个模式覆盖了绝大多数场景。核心原则是：保持 Composable 职责单一、返回值使用 ref 保持解构安全、必要时清理副作用。掌握这些模式后，你会发现代码复用变得自然而优雅。