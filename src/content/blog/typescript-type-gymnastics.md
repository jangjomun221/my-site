---
title: "TypeScript 类型体操入门：从 Utility Types 到自定义类型推导"
description: "系统梳理 TypeScript 高级类型技巧，涵盖条件类型、infer 关键字、模板字面量类型等，帮助你写出更安全的类型定义。"
pubDate: 2026-05-28
---

TypeScript 的类型系统是图灵完备的，这意味着你可以在类型层面做几乎任何计算。虽然日常开发不需要过于复杂的类型体操，但掌握核心技巧能显著提升代码的类型安全性。

## 基础：内置 Utility Types

TypeScript 内置了很多实用类型，先确保你熟练使用它们：

```typescript
// Partial —— 所有属性变为可选
type Draft = Partial<Article>

// Required —— 所有属性变为必需
type Complete = Required<Draft>

// Pick —— 选取部分属性
type Summary = Pick<Article, 'title' | 'date'>

// Omit —— 排除部分属性
type CreateInput = Omit<Article, 'id' | 'createdAt'>

// Record —— 构建键值对类型
type StatusMap = Record<'active' | 'inactive', User[]>
```

## 条件类型

条件类型是类型体操的核心，语法类似三元表达式：

```typescript
type IsString<T> = T extends string ? true : false

type A = IsString<'hello'>  // true
type B = IsString<42>       // false
```

结合分布式条件类型，可以对联合类型的每个成员独立运算：

```typescript
type NonNullable<T> = T extends null | undefined ? never : T

type C = NonNullable<string | null | undefined>  // string
```

## infer 关键字

`infer` 用于在条件类型中提取（推导）某个位置的类型：

```typescript
// 提取函数返回值类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never

// 提取 Promise 内部类型
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T

// 提取数组元素类型
type ElementOf<T> = T extends (infer E)[] ? E : never
```

`infer` 的强大之处在于它能在模式匹配中「捕获」类型变量。

## 模板字面量类型

TypeScript 4.1 引入的模板字面量类型让字符串级别的类型操作成为可能：

```typescript
type EventName<T extends string> = `on${Capitalize<T>}`

type ClickEvent = EventName<'click'>  // 'onClick'
type FocusEvent = EventName<'focus'>  // 'onFocus'
```

结合映射类型，可以批量生成事件处理器类型：

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
}

interface Person { name: string; age: number }
type PersonGetters = Getters<Person>
// { getName: () => string; getAge: () => number }
```

## 递归类型

处理嵌套结构时，递归类型不可或缺：

```typescript
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object
    ? DeepReadonly<T[K]>
    : T[K]
}

type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object
    ? DeepPartial<T[K]>
    : T[K]
}
```

## 实战：类型安全的事件系统

综合运用以上技巧，实现一个类型安全的事件总线：

```typescript
type EventMap = {
  userLogin: { userId: string; timestamp: number }
  pageView: { path: string; referrer?: string }
  error: { code: number; message: string }
}

type EventHandler<T> = (payload: T) => void

class TypedEventBus<Events extends Record<string, any>> {
  private handlers = new Map<string, Set<Function>>()

  on<K extends keyof Events>(
    event: K,
    handler: EventHandler<Events[K]>
  ) {
    if (!this.handlers.has(event as string)) {
      this.handlers.set(event as string, new Set())
    }
    this.handlers.get(event as string)!.add(handler)
  }

  emit<K extends keyof Events>(event: K, payload: Events[K]) {
    this.handlers.get(event as string)?.forEach(h => h(payload))
  }
}

const bus = new TypedEventBus<EventMap>()
bus.on('userLogin', ({ userId }) => console.log(userId))  // 类型正确
bus.emit('error', { code: 500, message: 'fail' })         // 类型正确
bus.emit('error', { wrong: true })                        // 编译报错
```

## 建议

类型体操虽然强大，但要克制使用。简单的 `as` 断言有时比复杂的类型推导更务实。团队中如果只有你能看懂这些类型，那可能就不该用。好的类型设计应该是让使用者感觉自然、IDE 提示清晰，而不是让维护者头疼。