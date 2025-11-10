### 1.SOP/SSR渲染流程

#### 1️⃣ SPA 首屏加载流程（传统客户端渲染）

假设用户访问 `/users` 页面，需要从接口获取用户列表：

1. 浏览器请求页面 → 后端返回一个**空 HTML** + JS 脚本文件
2. 浏览器下载 JS 文件
3. 浏览器执行 JS → 初始化 Vue 应用
4. Vue setup 执行 `$fetch('/api/users')` → 发请求给后端获取数据
5. 后端返回 API 数据 → Vue 渲染 DOM
6. 用户看到完整页面

**特点：**
- 首屏内容在 API 返回之前不可见 → **首屏白屏时间长**
- 如果接口多或慢，FCP（First Contentful Paint）会很晚
- 优点：服务端压力小，不需要 SSR

#### 2️⃣ Nuxt SSR（服务端渲染）首屏加载流程

同样访问 `/users` 页面：

1. 浏览器请求页面 URL → 请求到 Nuxt 服务端
2. 服务端运行 Vue 组件和页面逻辑
    - 执行 `useAsyncData` / `$fetch` → 请求后端 API 获取用户列表
    - 数据请求完成后，服务端把组件渲染成完整 HTML      
3. 服务端把完整 HTML 返回给浏览器 → 浏览器立即显示页面
4. 浏览器下载 JS 文件 → 执行 Hydration，把 DOM “激活”为 SPA 
5. Vue 挂载完成，用户可以继续操作交互
    

**特点：**
- 首屏内容**立即可见** → FCP 快
- 服务端做了数据请求和渲染 → 用户无需等待 JS 执行 API
- Hydration 只是激活 Vue 交互，不改变已渲染内容
- 如果请求过多，服务端压力大，但可以通过并行请求和缓存优化

### 2.代码优化
#### 1.组件添加 lazy
```vue
<script setup lang="ts">
const show = ref(false)
</script>

<template>
  <div>
    <h1>Mountains</h1>
    <LazyMountainsList v-if="show" />
    <button
      v-if="!show"
      @click="show = true"
    >
      Show List
    </button>
  </div>
</template>

```

- 不加 Lazy：组件的 JS 会被包含在当前页面的主 bundle 中，页面一加载就下载解析；v-if 只是让它不渲染（不挂载），但代码早就到客户端了。
- 加了 Lazy：组件会被拆成独立的异步 chunk，只有当 v-if 变为 true（组件需要渲染）时，才会发起网络请求下载对应 JS。
### 3.Nuxt 生命周期

Nuxt 应用的生命周期大致分为：

1. 服务器端首屏渲染（SSR）
2. 客户端接管与激活（Hydration/CSR）
3. 路由切换与后续渲染（客户端导航）
4. 卸载与清理
#### 阶段 1：服务器端首屏渲染（SSR）

在 Node 环境生成 HTML 字符串，无浏览器、无真实 DOM。

- 会发生（可用）
    - 创建应用与路由解析（匹配当前 URL）
    - 组件实例创建，执行 setup() / <script setup> 顶层同步代码
    - 响应式数据初始化（ref/reactive/computed）
    - 数据预取与注水
        - useAsyncData / useFetch（等待结果并序列化注入到 HTML）
        - onServerPrefetch（仅 SSR）
    - Nuxt 插件与中间件（服务器端分支）
        - defineNuxtPlugin 的服务端逻辑
        - 服务器端路由中间件（defineNuxtRouteMiddleware）
        - Nitro server/plugins、server/api 处理
- 不会发生（禁止/无效）
    - onBeforeMount / onMounted
    - onBeforeUpdate / onUpdated
    - onBeforeUnmount / onUnmounted
    - 任何 DOM/浏览器 API（window、document、Element、ResizeObserver 等）
    - 客户端事件与交互
- 备注
    - 顶层副作用需避免浏览器 API；若需客户端再跑一次，请用 process.server / import.meta.server 做分支
    - SSR 的 useAsyncData 结果会注水，客户端可直接复用

---

#### 阶段 2：客户端接管与激活（Hydration / 首次 CSR）

浏览器加载 SSR 的 HTML，Nuxt/Vue 在客户端创建同构树并激活事件。

- 会发生
    - 再次创建组件并执行 setup 顶层同步代码（同构要求，注意幂等）
    - 首次挂载生命周期：onBeforeMount → onMounted（可安全访问 DOM）
    - 客户端中间件与插件分支执行
    - 反序列化 SSR 注水数据（useAsyncData 直接可用，通常无需二次请求）
- 不会发生
    - onServerPrefetch（仅 SSR）
    - 服务器端专属逻辑（server/plugins、Nitro API 执行本身）
- 备注
    - 若在顶层启动计时器/订阅，请确保不会因“SSR 一次 + 客户端一次”而重复；用 process.client / import.meta.client 控制

---

#### 阶段 3：客户端路由切换与后续渲染（导航后 CSR）

SPA 模式下的页面切换与交互，全部在浏览器内进行。

- 会发生
    - 路由解析与中间件（客户端分支）
    - 新页面组件创建并执行 setup 顶层同步代码
    - 数据获取（策略化）
        - useAsyncData / useFetch（默认客户端请求，可结合缓存/SWR）
    - 组件生命周期
        - 新挂载：onBeforeMount / onMounted
        - 响应式更新：onBeforeUpdate / onUpdated
    - keep-alive 相关：onActivated / onDeactivated（若使用）
- 不会发生
    - SSR 渲染与 onServerPrefetch（除非整页刷新触发新一轮 SSR）
- 备注
    - SEO 依赖首屏 SSR；后续多为客户端动态更新

---

#### 阶段 4：卸载与清理（仅客户端）

组件离开路由或被销毁时的清理阶段。

- 会发生
    - onBeforeUnmount / onUnmounted
    - 清理订阅、定时器、事件监听、第三方实例
- 不会发生
    - 任何 SSR 专属逻辑
- 备注
    - 与 keep-alive 配合时，频繁进入/离开使用 onActivated/onDeactivated 管理资源而非销毁重建

---

#### 环境对照速查表

- setup 顶层同步代码：SSR 运行；客户端激活时再次运行；客户端新组件仍会运行
- onServerPrefetch：仅 SSR
- useAsyncData / useFetch：SSR 等待并注水；客户端激活复用；客户端导航按策略请求
- 挂载/更新/卸载类钩子（onMounted 等）：仅客户端
- DOM/浏览器 API：仅客户端（onMounted 及之后更安全）

环境判断键：

- 仅服务器端：process.server 或 import.meta.server
- 仅客户端：process.client 或 import.meta.client