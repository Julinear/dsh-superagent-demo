# findu 校园失物招领平台 — 架构设计文档

> 本文档系统阐述 findu 平台的前端架构设计，涵盖整体架构、目录结构、模块职责、数据流、状态管理、路由设计、接口层、组件体系与扩展策略，为开发者提供全局视角的设计参考。

---

## 一、设计目标

| 目标         | 说明                                                                 |
| ------------ | ------------------------------------------------------------------- |
| 可维护性     | 按业务模块拆分目录，职责清晰，降低耦合，便于团队协作与长期演进。     |
| 类型安全     | 全面启用 TypeScript `strict` 模式，从接口到视图全链路类型覆盖。     |
| 可复用性     | 通过组合式函数（composables）与通用组件抽象，最大化逻辑与 UI 复用。  |
| 可扩展性     | 模块化设计，新业务模块可按既定模式快速接入，不破坏既有结构。         |
| 一致性       | 统一的编码规范、命名约定与提交规范，保证代码风格与协作节奏一致。   |
| 可追溯性     | 物品状态流转全程留痕，Git 提交遵循 Conventional Commits，便于审计。 |

---

## 二、整体架构

findu 采用 **前端单体应用 + 分层架构** 的组织方式，以 Vue 3 为核心，围绕「视图层 — 状态层 — 服务层 — 工具层」四层结构展开：

```
┌─────────────────────────────────────────────────────────────┐
│                        浏览器 (Browser)                      │
├─────────────────────────────────────────────────────────────┤
│  ┌────────────────────────────────────────────────────────┐  │
│  │                     视图层 (Views)                     │  │
│  │  页面级组件，按业务模块组织                              │  │
│  │  lost / found / search / profile / message             │  │
│  └────────────────┬───────────────────────────────────────┘  │
│                   │ 读写状态、调用 API                         │
│  ┌────────────────▼───────────────────────────────────────┐  │
│  │                   状态层 (Pinia Stores)                 │  │
│  │  user / item / message / category                     │  │
│  │  集中管理跨组件共享状态                                  │  │
│  └────────────────┬───────────────────────────────────────┘  │
│                   │ 发起请求                                   │
│  ┌────────────────▼───────────────────────────────────────┐  │
│  │                   服务层 (API)                          │  │
│  │  按业务模块拆分的接口函数                                │  │
│  │  统一 Axios 实例 + 拦截器                               │  │
│  └────────────────┬───────────────────────────────────────┘  │
│                   │ HTTP                                      │
│  ┌────────────────▼───────────────────────────────────────┐  │
│  │                   后端服务 (RESTful API)                │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              工具层 (Utils / Composables)              │  │
│  │  请求封装、格式化、校验、组合式函数                       │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 架构分层说明

| 层级     | 目录            | 职责                                                                 |
| -------- | --------------- | ------------------------------------------------------------------- |
| 视图层   | `src/views/`    | 页面级组件，负责 UI 渲染与用户交互，组合通用组件与状态。             |
| 组件层   | `src/components/` | 全局通用组件，与具体业务解耦，可跨页面复用。                       |
| 状态层   | `src/stores/`   | Pinia Store，管理跨组件共享的应用状态，提供 actions 修改状态。      |
| 服务层   | `src/api/`      | 按业务模块组织的接口函数，调用统一的 Axios 实例发起请求。            |
| 工具层   | `src/utils/`、`src/composables/` | 纯工具函数与可复用的组合式逻辑。                        |
| 路由层   | `src/router/`   | 集中式路由配置，支持懒加载与导航守卫。                               |
| 样式层   | `src/styles/`   | 全局样式、变量与 mixin，提供统一设计令牌。                           |

---

## 三、目录结构详解

```
findu/
├── public/                    # 静态资源（直接拷贝，不经过构建处理）
│   └── favicon.ico
├── src/
│   ├── api/                   # 接口请求，按业务模块拆分
│   │   ├── item.ts            # 失物/拾物相关接口
│   │   ├── user.ts            # 用户认证与个人信息接口
│   │   ├── message.ts         # 站内消息接口
│   │   ├── category.ts        # 分类接口
│   │   └── favorite.ts       # 收藏接口
│   ├── assets/                # 需经构建处理的静态资源（图片、字体、样式等）
│   ├── components/            # 全局通用组件
│   │   ├── ItemCard.vue       # 物品卡片
│   │   ├── CategoryFilter.vue # 分类筛选器
│   │   ├── SearchBar.vue      # 搜索栏
│   │   ├── StatusBadge.vue    # 状态标签
│   │   ├── EmptyState.vue     # 空状态占位
│   │   ├── ImageUploader.vue  # 图片上传组件
│   │   └── Pagination.vue     # 分页器
│   ├── composables/           # 组合式函数（可复用逻辑）
│   │   ├── use-item.ts        # 物品列表/详情逻辑
│   │   ├── use-search.ts      # 搜索与筛选逻辑
│   │   ├── use-message.ts     # 消息轮询/订阅逻辑
│   │   ├── use-favorite.ts    # 收藏逻辑
│   │   └── use-category.ts    # 分类树逻辑
│   ├── layouts/               # 布局组件
│   │   ├── DefaultLayout.vue  # 默认布局（头部 + 内容 + 底部）
│   │   └── BlankLayout.vue     # 空白布局（登录/注册页）
│   ├── router/                # 路由配置
│   │   └── index.ts           # 路由表 + 导航守卫
│   ├── stores/                # Pinia 状态管理
│   │   ├── user.ts            # 用户状态
│   │   ├── item.ts            # 物品列表与详情状态
│   │   ├── message.ts         # 消息状态
│   │   ├── category.ts        # 分类状态
│   │   └── favorite.ts        # 收藏状态
│   ├── styles/                # 全局样式与变量
│   │   ├── variables.scss     # 设计令牌（颜色、间距、字号）
│   │   ├── mixins.scss        # 通用 mixin
│   │   └── global.scss        # 全局重置与基础样式
│   ├── types/                 # 公共类型定义
│   │   ├── item.ts            # 物品相关类型
│   │   ├── user.ts            # 用户相关类型
│   │   ├── message.ts         # 消息相关类型
│   │   ├── category.ts        # 分类相关类型
│   │   └── api.ts             # API 响应封装类型
│   ├── utils/                 # 工具函数
│   │   ├── request.ts         # Axios 实例封装与拦截器
│   │   ├── format.ts          # 格式化（日期、状态文案等）
│   │   ├── validate.ts        # 表单校验
│   │   └── storage.ts         # 本地存储封装
│   ├── views/                 # 页面级组件，按业务模块组织
│   │   ├── home/              # 首页
│   │   ├── lost/              # 寻物启事模块
│   │   │   ├── LostList.vue
│   │   │   ├── LostDetail.vue
│   │   │   └── LostPublish.vue
│   │   ├── found/             # 拾物信息模块
│   │   │   ├── FoundList.vue
│   │   │   ├── FoundDetail.vue
│   │   │   └── FoundPublish.vue
│   │   ├── search/           # 搜索结果页
│   │   ├── profile/           # 个人中心
│   │   │   ├── ProfileIndex.vue
│   │   │   ├── MyItems.vue
│   │   │   ├── MyFavorites.vue
│   │   │   └── MyMessages.vue
│   │   └── auth/              # 登录/注册
│   │       ├── Login.vue
│   │       └── Register.vue
│   ├── App.vue                # 根组件
│   └── main.ts                # 应用入口
├── .editorconfig
├── .eslintrc.cjs
├── .prettierrc
├── index.html
├── package.json
├── tsconfig.json
└── vite.config.ts
```

---

## 四、核心数据流

findu 采用 **单向数据流** 模型，数据从后端经服务层流入状态层，再由视图层消费：

```
┌──────────┐     HTTP      ┌──────────┐    dispatch     ┌──────────┐
│  后端 API │ ──────────▶  │  API 层  │ ─────────────▶ │  Store   │
└──────────┘               └──────────┘                 └────┬─────┘
                                                            │ subscribe
                               ┌──────────┐                 ▼
                               │  视图层   │ ◀────────── ┌──────────┐
                               │ (Views)  │   render     │  Components│
                               └────┬─────┘              └──────────┘
                                    │ user action
                                    ▼
                              ┌──────────┐
                              │  Store   │ ──▶ action ──▶ API 层 ──▶ 后端
                              └──────────┘
```

### 交互流程示例：发布寻物启事

1. 用户在 `LostPublish.vue` 填写表单并提交。
2. 视图调用 `useItem` composable 的 `publishLostItem` 方法。
3. composable 调用 `itemStore.publishLostItem()` action。
4. Store action 调用 `api/item.ts` 中的 `createLostItem` 接口。
5. API 层通过 `utils/request.ts` 封装的 Axios 实例发起 POST 请求。
6. 响应拦截器统一处理后，返回数据交回 Store。
7. Store 更新 `items` 列表状态。
8. 视图通过响应式订阅自动更新列表。

---

## 五、状态管理设计

采用 **Pinia** 作为状态管理方案，按业务域拆分 Store，各 Store 之间相互独立、按需组合。

### Store 划分

| Store           | 文件              | 管理状态                                   | 核心 Actions                                |
| --------------- | ----------------- | ------------------------------------------ | ------------------------------------------- |
| useUserStore    | `stores/user.ts`  | 登录态、用户信息、token                     | login, logout, fetchProfile, updateProfile   |
| useItemStore    | `stores/item.ts`  | 物品列表、当前详情、筛选条件、分页           | fetchItems, fetchDetail, publishItem, updateStatus, editItem, archiveItem |
| useMessageStore | `stores/message.ts` | 消息列表、未读数、消息筛选                | fetchMessages, markAsRead, markAllRead, startPolling, stopPolling |
| useCategoryStore | `stores/category.ts` | 分类树、当前选中分类                     | fetchCategories, selectCategory             |
| useFavoriteStore | `stores/favorite.ts` | 收藏列表、收藏状态                         | fetchFavorites, toggleFavorite, removeFavorite |

### 设计原则

- **单一数据源**：每个业务域的状态集中在一个 Store 中，组件不持有跨域状态。
- **Action 驱动**：状态修改必须通过 Store 的 action，禁止组件直接改写 `state`。
- **Getter 派生**：需要计算或过滤的状态通过 `getters` 派生，避免冗余存储。
- **持久化策略**：`useUserStore` 的 token 与「记住我」选项通过 `localStorage` 持久化；其余状态按需从服务端拉取，不持久化。
- **轮询生命周期**：`useMessageStore` 的轮询需配合页面可见性感知，避免后台无效请求。

### Store 结构示例

```typescript
// stores/item.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import * as itemApi from '@/api/item'
import type { ItemInfo, ItemStatus, ItemQuery } from '@/types/item'

export const useItemStore = defineStore('item', () => {
  // State
  const items = ref<ItemInfo[]>([])
  const currentDetail = ref<ItemInfo | null>(null)
  const query = ref<ItemQuery>({ status: 'all', category: '', keyword: '' })
  const loading = ref(false)

  // Getters
  const activeItems = computed(() =>
    items.value.filter(item => item.status !== 'archived')
  )

  // Actions
  async function fetchItems() {
    loading.value = true
    try {
      const res = await itemApi.getItemList(query.value)
      items.value = res.data
    } finally {
      loading.value = false
    }
  }

  async function publishItem(payload: Omit<ItemInfo, 'id'>) {
    const res = await itemApi.createLostItem(payload)
    items.value.unshift(res.data)
    return res.data
  }

  async function updateStatus(id: string, status: ItemStatus) {
    await itemApi.updateItemStatus(id, status)
    const target = items.value.find(i => i.id === id)
    if (target) target.status = status
  }

  async function editItem(id: string, data: Partial<ItemInfo>) {
    const res = await itemApi.updateItem(id, data)
    const target = items.value.find(i => i.id === id)
    if (target) Object.assign(target, res.data)
    return res.data
  }

  async function archiveItem(id: string) {
    await itemApi.archiveItem(id)
    const target = items.value.find(i => i.id === id)
    if (target) target.status = 'archived'
  }

  return { items, currentDetail, query, loading, activeItems, fetchItems, publishItem, updateStatus, editItem, archiveItem }
})
```

---

## 六、路由设计

路由采用 **Vue Router 4**，以懒加载 + 模块化组织，并配置导航守卫保护需鉴权的页面。

### 路由表概览

| 路径              | 页面组件              | 布局          | 鉴权 | 说明               |
| ----------------- | --------------------- | ------------- | ---- | ------------------ |
| `/`               | `home/HomeView.vue`   | DefaultLayout | 否   | 首页（物品概览）    |
| `/lost`           | `lost/LostList.vue`   | DefaultLayout | 否   | 寻物启事列表       |
| `/lost/:id`       | `lost/LostDetail.vue` | DefaultLayout | 否   | 寻物启事详情       |
| `/lost/publish`   | `lost/LostPublish.vue`| DefaultLayout | 是   | 发布寻物启事       |
| `/found`          | `found/FoundList.vue` | DefaultLayout | 否   | 拾物信息列表       |
| `/found/:id`      | `found/FoundDetail.vue`| DefaultLayout| 否   | 拾物信息详情       |
| `/found/publish`  | `found/FoundPublish.vue`| DefaultLayout| 是  | 发布拾物信息       |
| `/search`         | `search/SearchResult.vue`| DefaultLayout| 否 | 搜索结果页         |
| `/profile`        | `profile/ProfileIndex.vue`| DefaultLayout| 是 | 个人中心           |
| `/profile/items`  | `profile/MyItems.vue` | DefaultLayout | 是   | 我发布的物品       |
| `/profile/favorites` | `profile/MyFavorites.vue`| DefaultLayout| 是 | 我的收藏         |
| `/profile/messages` | `profile/MyMessages.vue`| DefaultLayout| 是 | 站内消息           |
| `/login`          | `auth/Login.vue`      | BlankLayout   | 否   | 登录               |
| `/register`       | `auth/Register.vue`   | BlankLayout   | 否   | 注册               |
| `/:pathMatch(.*)*` | `NotFound.vue`       | DefaultLayout | 否   | 404 页             |

### 导航守卫

```typescript
// router/index.ts
router.beforeEach((to, from) => {
  const userStore = useUserStore()
  if (to.meta.requiresAuth && !userStore.isLoggedIn) {
    return { name: 'Login', query: { redirect: to.fullPath } }
  }
})
```

### 懒加载策略

所有页面级组件均使用动态 `import()` 实现路由级代码分割，首屏仅加载首页与布局相关代码：

```typescript
const routes: RouteRecordRaw[] = [
  {
    path: '/lost',
    component: () => import('@/views/lost/LostList.vue'),
  },
  // ...
]
```

---

## 七、接口层设计

### Axios 封装

所有请求统一通过 `src/utils/request.ts` 中封装的 Axios 实例发起，统一处理鉴权、错误码与响应格式：

```typescript
// utils/request.ts
import axios from 'axios'
import { useUserStore } from '@/stores/user'

const request = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 15000,
})

// 请求拦截：携带 token
request.interceptors.request.use((config) => {
  const userStore = useUserStore()
  if (userStore.token) {
    config.headers.Authorization = `Bearer ${userStore.token}`
  }
  return config
})

// 响应拦截：统一错误处理
request.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response?.status === 401) {
      // token 失效，清除登录态并跳转登录
      const userStore = useUserStore()
      userStore.logout()
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default request
```

### API 模块拆分

接口按业务模块拆分至 `src/api/`，每个模块导出具名函数，命名遵循「动词 + 资源」规范：

```typescript
// api/item.ts
import request from '@/utils/request'
import type { ItemInfo, ItemQuery, ItemStatus } from '@/types/item'
import type { ApiResponse, PaginatedData } from '@/types/api'

export function getItemList(params: ItemQuery): ApiResponse<PaginatedData<ItemInfo>> {
  return request.get('/items', { params })
}

export function getItemDetail(id: string): ApiResponse<ItemInfo> {
  return request.get(`/items/${id}`)
}

export function createLostItem(data: Omit<ItemInfo, 'id' | 'status'>): ApiResponse<ItemInfo> {
  return request.post('/items/lost', data)
}

export function createFoundItem(data: Omit<ItemInfo, 'id' | 'status'>): ApiResponse<ItemInfo> {
  return request.post('/items/found', data)
}

export function updateItem(id: string, data: Partial<ItemInfo>): ApiResponse<ItemInfo> {
  return request.patch(`/items/${id}`, data)
}

export function updateItemStatus(id: string, status: ItemStatus): ApiResponse<void> {
  return request.patch(`/items/${id}/status`, { status })
}

export function archiveItem(id: string): ApiResponse<void> {
  return request.patch(`/items/${id}/status`, { status: 'archived' })
}
```

```typescript
// api/favorite.ts
import request from '@/utils/request'
import type { ItemInfo } from '@/types/item'
import type { ApiResponse, PaginatedData } from '@/types/api'

export function getFavoriteList(params: { page: number; pageSize: number }): ApiResponse<PaginatedData<ItemInfo>> {
  return request.get('/favorites', { params })
}

export function addFavorite(itemId: string): ApiResponse<void> {
  return request.post('/favorites', { itemId })
}

export function removeFavorite(itemId: string): ApiResponse<void> {
  return request.delete(`/favorites/${itemId}`)
}
```

```typescript
// api/message.ts
import request from '@/utils/request'
import type { MessageInfo, MessageType } from '@/types/message'
import type { ApiResponse, PaginatedData } from '@/types/api'

export function getMessageList(params: { type?: MessageType; read?: boolean; page: number; pageSize: number }): ApiResponse<PaginatedData<MessageInfo>> {
  return request.get('/messages', { params })
}

export function getUnreadCount(): ApiResponse<number> {
  return request.get('/messages/unread-count')
}

export function markAsRead(id: string): ApiResponse<void> {
  return request.patch(`/messages/${id}/read`)
}

export function markAllAsRead(): ApiResponse<void> {
  return request.patch('/messages/read-all')
}
```

```typescript
// api/category.ts
import request from '@/utils/request'
import type { Category } from '@/types/category'
import type { ApiResponse } from '@/types/api'

export function getCategoryTree(): ApiResponse<Category[]> {
  return request.get('/categories/tree')
}
```

### 统一响应类型

```typescript
// types/api.ts
export interface ApiResponse<T = unknown> {
  code: number
  message: string
  data: T
}

export interface PaginatedData<T> {
  list: T[]
  total: number
  page: number
  pageSize: number
}
```

---

## 八、组件体系设计

### 组件分层

```
┌─────────────────────────────────────────────┐
│            页面组件 (src/views/)            │  ← 业务逻辑与页面编排
├─────────────────────────────────────────────┤
│  布局组件 (src/layouts/) + 通用组件 (src/components/) │  ← 结构骨架与可复用 UI
├─────────────────────────────────────────────┤
│        组合式函数 (src/composables/)         │  ← 可复用业务逻辑
└─────────────────────────────────────────────┘
```

### 通用组件清单

| 组件             | 职责                                         | 关键 Props                                           |
| ---------------- | -------------------------------------------- | --------------------------------------------------- |
| `ItemCard`       | 物品卡片，展示缩略图、标题、分类、状态、时间  | `item: ItemInfo`, `clickable: boolean`              |
| `CategoryFilter` | 分类筛选器，支持单选/多选                    | `categories: Category[]`, `modelValue: string[]`    |
| `SearchBar`      | 搜索栏，支持关键字输入与防抖                  | `modelValue: string`, `placeholder: string`        |
| `StatusBadge`    | 状态标签，根据物品状态显示对应颜色与文案      | `status: ItemStatus`                                |
| `EmptyState`     | 空状态占位，支持自定义图标与文案              | `description: string`, `image?: string`            |
| `ImageUploader`  | 图片上传组件，支持预览、压缩与大小限制        | `modelValue: string[]`, `maxSize: number`           |
| `Pagination`     | 分页器                                       | `total: number`, `current: number`, `pageSize: number` |

### 组件通信策略

| 场景         | 方式                          | 示例                                |
| ------------ | ----------------------------- | ----------------------------------- |
| 父 → 子      | `props`                       | `<ItemCard :item="data" />`         |
| 子 → 父      | `emits`                       | `@click="onItemClick"`              |
| 跨层级        | `provide` / `inject`          | 布局向子页面注入用户信息             |
| 全局共享      | Pinia Store                   | 用户登录态、物品列表                 |
| 临时通信      | composables 内部 ref          | 搜索框防抖状态                       |

---

## 九、类型系统设计

### 核心业务类型

```typescript
// types/item.ts
export type ItemType = 'lost' | 'found'

export type ItemStatus = 'searching' | 'found' | 'returned' | 'pending' | 'archived'

export interface ItemInfo {
  id: string
  type: ItemType
  title: string
  description: string
  category: string               // 分类 ID
  categoryName?: string           // 分类名称（后端冗余，减少前端映射）
  location: string
  occurredAt: string             // 丢失 / 捡拾时间 (ISO 8601)
  images: string[]                // 图片 URL 列表
  status: ItemStatus
  publisherId: string
  publisherName: string
  publisherAvatar?: string
  contact: string
  contactVisible: boolean        // 是否公开联系方式
  isDeleted: boolean              // 软删除标记
  favoriteCount: number           // 收藏数
  createdAt: string
  updatedAt: string
}

export interface ItemQuery {
  type?: ItemType
  status?: ItemStatus | 'all'
  category?: string
  keyword?: string
  page?: number
  pageSize?: number
}

export interface FavoriteRecord {
  id: string
  userId: string
  itemId: string
  createdAt: string
}
```

```typescript
// types/category.ts
export interface Category {
  id: string
  name: string
  parentId: string | null
  icon?: string                  // 分类图标 URL 或图标名
  sort: number                   // 排序权重
  children?: Category[]          // 子分类
}
```

```typescript
// types/user.ts
export interface UserInfo {
  id: string
  username: string
  nickname: string
  avatar: string
  phone?: string
  email?: string
  studentId?: string
}

export interface LoginParams {
  username: string
  password: string
}

export interface LoginResult {
  token: string
  user: UserInfo
}
```

```typescript
// types/message.ts
export type MessageType = 'clue' | 'status' | 'system'

export interface MessageInfo {
  id: string
  type: MessageType
  title: string                  // 消息标题
  content: string                // 消息正文
  itemId?: string                // 关联物品 ID（线索 / 状态消息）
  receiverId: string             // 接收者 ID
  senderId?: string              // 发送者 ID（系统消息为空）
  read: boolean
  readAt?: string                // 已读时间
  createdAt: string
}
```

### 状态流转类型

```
寻物启事状态流转:
  searching ──→ found ──→ archived
      │
      └──→ archived (下架 / 超时归档)

拾物信息状态流转:
  pending ──→ returned ──→ archived
      │
      └──→ archived (下架 / 超时归档)
```

```typescript
// types/item.ts (续)

// 合法状态转换关系
const VALID_TRANSITIONS: Record<ItemStatus, ItemStatus[]> = {
  searching: ['found', 'archived'],
  found:     ['archived'],
  pending:   ['returned', 'archived'],
  returned:  ['archived'],
  archived:  [],  // 终态，不可再变更
}

export function canTransition(from: ItemStatus, to: ItemStatus): boolean {
  return VALID_TRANSITIONS[from]?.includes(to) ?? false
}
```

### 自动归档规则

后端定时任务负责将超时物品自动归档，前端无需处理，但需响应归档后的状态同步：

| 条件                  | 动作                          |
| --------------------- | ----------------------------- |
| `searching` 超过 30 天 | 自动转 `archived`，通知发布者  |
| `pending` 超过 30 天   | 自动转 `archived`，通知发布者  |
| `found` 超过 7 天      | 自动转 `archived`             |
| `returned` 超过 7 天   | 自动转 `archived`             |

---

## 十、组合式函数设计

composables 封装可复用的业务逻辑，使视图组件保持轻量：

| composable        | 职责                                             | 返回                                   |
| ----------------- | ------------------------------------------------ | -------------------------------------- |
| `useItem`         | 物品列表查询、详情获取、发布、状态变更、编辑、下架  | items, detail, loading, actions         |
| `useSearch`       | 关键字搜索、防抖、筛选条件管理                    | keyword, results, filters, debouncedSearch |
| `useMessage`      | 消息列表拉取、未读计数、标记已读、轮询控制          | messages, unreadCount, markAsRead, startPolling, stopPolling |
| `usePagination`   | 分页状态管理                                      | currentPage, total, hasNext, next, prev |
| `useImageUpload`  | 图片选择、压缩、上传                              | images, uploading, upload, remove      |
| `useAuth`         | 登录态判断、权限守卫                              | isLoggedIn, currentUser, requireAuth   |
| `useFavorite`     | 收藏 / 取消收藏、收藏状态管理                      | favorites, isFavorited, toggleFavorite |
| `useCategory`     | 分类树拉取、分类选中状态管理                       | categories, selectedCategory, selectCategory |

### 示例：useSearch

```typescript
// composables/use-search.ts
import { ref, watch } from 'vue'
import { useItemStore } from '@/stores/item'
import type { ItemQuery } from '@/types/item'

export function useSearch(initialQuery: Partial<ItemQuery> = {}) {
  const itemStore = useItemStore()
  const keyword = ref(initialQuery.keyword ?? '')
  const category = ref(initialQuery.category ?? '')
  const status = ref(initialQuery.status ?? 'all')

  let debounceTimer: ReturnType<typeof setTimeout> | null = null

  function debouncedSearch() {
    if (debounceTimer) clearTimeout(debounceTimer)
    debounceTimer = setTimeout(() => {
      itemStore.query.keyword = keyword.value
      itemStore.query.category = category.value
      itemStore.query.status = status.value
      itemStore.fetchItems()
    }, 300)
  }

  watch([keyword, category, status], debouncedSearch)

  return { keyword, category, status }
}
```

---

## 十一、构建与部署架构

### 构建工具

使用 **Vite** 作为构建工具，利用其基于 ES 模块的极速冷启动与 HMR 能力。

### 环境变量

| 变量                  | 说明                     | 示例                          |
| --------------------- | ------------------------ | ----------------------------- |
| `VITE_API_BASE_URL`   | 后端 API 基础地址         | `https://api.findu.edu.cn`    |
| `VITE_APP_TITLE`      | 应用标题                 | `findu 校园失物招领平台`       |
| `VITE_UPLOAD_MAX_SIZE`| 图片上传大小上限（字节） | `5242880`                     |

通过 `.env.development` 与 `.env.production` 区分开发与生产环境：

```bash
# .env.development
VITE_API_BASE_URL=http://localhost:3000/api
VITE_APP_TITLE=findu (Dev)

# .env.production
VITE_API_BASE_URL=https://api.findu.edu.cn
VITE_APP_TITLE=findu 校园失物招领平台
```

### 代码分割策略

```
入口 (main.ts)
  ├── vendor chunk:    vue, vue-router, pinia, axios
  ├── layout chunk:    DefaultLayout, BlankLayout
  ├── views/lost/      chunk: LostList, LostDetail, LostPublish
  ├── views/found/     chunk: FoundList, FoundDetail, FoundPublish
  ├── views/profile/   chunk: ProfileIndex, MyItems, MyFavorites, MyMessages
  └── views/auth/      chunk: Login, Register
```

Vite `build.rollupOptions.output.manualChunks` 将第三方依赖与业务代码分离，优化缓存命中率。

### 部署方式

```
┌──────────────┐   pnpm build   ┌──────────────┐   部署    ┌──────────────┐
│  源码仓库    │ ─────────────▶ │ dist/ 产物   │ ────────▶ │  CDN / Nginx  │
└──────────────┘               └──────────────┘           └──────────────┘
                                                             │
                                                   静态资源 + index.html
                                                   SPA History 模式 fallback
```

- 构建产物为纯静态文件（`dist/`），部署至 CDN 或静态服务器。
- Nginx 配置 `try_files $uri $uri/ /index.html` 以支持 History 路由模式。

---

## 十二、代码质量保障体系

```
┌──────────────────────────────────────────────────────────────┐
│                      代码质量保障链路                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  开发阶段                                                    │
│  ├── TypeScript strict ──▶ 编译期类型检查                     │
│  ├── ESLint ────────────▶ 代码规范检查                        │
│  ├── Prettier ──────────▶ 代码格式化                         │
│  └── Stylelint ─────────▶ 样式规范检查                        │
│                                                              │
│  提交阶段                                                    │
│  ├── Husky pre-commit ──▶ lint-staged (ESLint + Prettier)    │
│  └── Husky commit-msg ──▶ commitlint (Conventional Commits)  │
│                                                              │
│  CI 阶段                                                     │
│  ├── 类型检查 (vue-tsc)                                      │
│  ├── Lint 检查                                               │
│  ├── 单元测试 (Vitest)                                       │
│  └── 构建验证 (pnpm build)                                   │
│                                                              │
│  Review 阶段                                                 │
│  └── Code Review ──▶ 合并至 develop / main                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 十三、安全设计

| 方面           | 措施                                                         |
| -------------- | ------------------------------------------------------------ |
| 鉴权           | JWT token，请求拦截器自动携带 `Authorization` Header         |
| Token 失效     | 响应拦截器捕获 401，清除登录态并跳转登录页                   |
| XSS 防护       | Vue 模板默认转义，禁止使用 `v-html` 渲染用户输入              |
| 输入校验       | 表单提交前进行前端校验（必填、格式、长度），后端二次校验      |
| 图片上传       | 限制文件类型与大小（`VITE_UPLOAD_MAX_SIZE`），前端压缩后上传  |
| 敏感信息       | token 仅存储于 `localStorage`，不暴露用户密码等敏感字段      |
| CSRF           | 后端校验 `Origin` / `Referer`，关键操作使用 POST/PATCH 方法  |

---

## 十四、性能优化策略

| 策略             | 实现方式                                                     |
| ---------------- | ------------------------------------------------------------ |
| 路由懒加载       | 页面级组件动态 `import()`，按需加载                          |
| 第三方拆包       | `manualChunks` 将 vendor 与业务代码分离，提升缓存命中率       |
| 图片懒加载       | `IntersectionObserver` 或 `loading="lazy"` 实现列表图片懒加载 |
| 搜索防抖         | `useSearch` composable 中 300ms 防抖，减少无效请求           |
| 收藏防抖         | `useFavorite` 中 500ms 防抖，合并收藏 / 取消操作              |
| 列表虚拟滚动     | 大量物品列表采用虚拟滚动，仅渲染可视区域                     |
| 请求缓存         | 分类等低频变更数据首次拉取后缓存在 Store 中                   |
| 消息轮询优化      | 页面不可见时暂停轮询，可见时恢复，减少后台请求               |
| 代码压缩         | Vite 生产构建默认启用 Terser 压缩 + CSS 压缩                 |
| Gzip / Brotli   | 部署服务器开启压缩传输                                       |

---

## 十五、扩展性设计

### 新增业务模块的步骤

以新增「公告模块」为例：

1. **类型定义**：在 `types/` 下新增 `announcement.ts`。
2. **API 模块**：在 `api/` 下新增 `announcement.ts`。
3. **状态管理**：在 `stores/` 下新增 `announcement.ts`（Pinia Store）。
4. **页面组件**：在 `views/announcement/` 下新增页面组件。
5. **路由配置**：在 `router/index.ts` 中注册新路由。
6. **通用组件**：如需复用 UI，在 `components/` 下新增。
7. **composable**：如需复用逻辑，在 `composables/` 下新增。

整个流程遵循既有目录约定与分层模式，无需修改框架层代码。

### 设计约束

- **视图层不直接调用 API**：必须经 Store action 或 composable 调用，保证状态可追溯。
- **通用组件不依赖 Store**：保持纯 UI 层，数据由调用方通过 props 注入。
- **类型优先**：任何新增数据结构先在 `types/` 定义，再在各层引用。
- **单一职责**：每个文件、组件、Store 只承担一个明确的职责。

---

## 十六、消息通知架构

### 通知触发机制

消息由后端在特定业务事件发生时生成，前端通过轮询拉取：

```
┌──────────────┐                        ┌──────────────┐
│  用户操作     │                        │  后端服务     │
├──────────────┤                        ├──────────────┤
│ 发布拾物信息  │──匹配到寻物启事──▶     │ 生成线索消息  │
│ 变更物品状态  │───────────────────▶     │ 生成状态消息  │
│ 管理员操作    │───────────────────▶     │ 生成系统消息  │
│ 定时归档任务  │───────────────────▶     │ 生成状态消息  │
└──────────────┘                        └──────┬───────┘
                                               │ 推送
                                               ▼
                                        ┌──────────────┐
                                        │  前端消息中心  │
                                        │  (轮询拉取)    │
                                        └──────────────┘
```

### 消息类型

| 类型       | `type` 值  | 触发场景                                   |
| ---------- | ---------- | ------------------------------------------ |
| 线索消息   | `clue`     | 他人发布了可能匹配的拾物 / 寻物信息          |
| 状态消息   | `status`   | 收藏或关联的物品状态变更、自动归档           |
| 系统消息   | `system`   | 平台公告、违规提醒、账号相关                 |

### 轮询策略

当前阶段采用轮询方案，保证实现简单可靠；未来可扩展为 WebSocket 实时推送。

| 策略               | 实现                                                     |
| ------------------ | -------------------------------------------------------- |
| 轮询间隔            | 每 30 秒拉取一次未读消息数（`GET /messages/unread-count`） |
| 页面可见性感知       | 监听 `document.visibilitychange`，页面不可见时暂停轮询    |
| 页面恢复时立即拉取   | 页面重新可见时立即发起一次拉取，再恢复正常轮询             |
| 进入消息中心         | 拉取完整消息列表（`GET /messages`），停止未读数轮询       |
| 离开消息中心         | 恢复未读数轮询                                            |
| 轮询控制            | `useMessageStore` 的 `startPolling` / `stopPolling` action |

### 消息与状态变更联动

物品状态变更（手动或自动归档）触发状态消息通知关联用户：

```
物品状态变更 ──▶ 后端生成状态消息 ──▶ 前端轮询拉取 ──▶ 更新未读角标
                                                    ──▶ 用户点击查看 ──▶ 标记已读
                                                    ──▶ 点击线索消息 ──▶ 跳转物品详情
```

---

## 十七、关键流程时序图

### 失物发布与匹配流程

```
失主              前端应用            API层            后端服务           拾到者
 │                  │                  │                │                  │
 │  发布寻物启事    │                  │                │                  │
 │─────────────────▶│                  │                │                  │
 │                  │  publishItem()   │                │                  │
 │                  │─────────────────▶│                │                  │
 │                  │                  │  POST /items   │                  │
 │                  │                  │───────────────▶│                  │
 │                  │                  │                │ 存储，状态=searching
 │                  │                  │  201 Created  │                  │
 │                  │                  │◀───────────────│                  │
 │                  │  返回物品信息     │                │                  │
 │                  │◀─────────────────│                │                  │
 │  显示发布成功    │                  │                │                  │
 │◀─────────────────│                  │                │                  │
 │                  │                  │                │                  │
 │                  │                  │                │  发布拾物信息     │
 │                  │                  │                │◀─────────────────│
 │                  │                  │                │ 匹配到寻物启事    │
 │                  │                  │                │──────────────────▶│
 │                  │                  │                │ 发送站内消息      │
 │                  │  消息通知         │                │                  │
 │◀─────────────────│◀─────────────────│◀───────────────│                  │
 │  查看消息        │                  │                │                  │
 │─────────────────▶│                  │                │                  │
 │                  │  线下沟通确认     │                │                  │
 │◀─────────────────│                  │                │                  │
 │  更新状态=found  │                  │                │                  │
 │─────────────────▶│  updateStatus() │                │                  │
 │                  │─────────────────▶│  PATCH /status │                  │
 │                  │                  │───────────────▶│                  │
```

---

## 十八、技术选型汇总

| 类别         | 选型               | 选型理由                                           |
| ------------ | ------------------ | -------------------------------------------------- |
| 框架         | Vue 3 (Composition API) | 组合式 API 利于逻辑复用，响应式系统高效            |
| 语言         | TypeScript         | 静态类型检查，提升可维护性与开发体验                |
| 构建工具     | Vite               | 极速 HMR 与构建，原生 ES 模块支持                  |
| 状态管理     | Pinia              | 轻量、类型友好、DevTools 集成，Vue 官方推荐        |
| 路由         | Vue Router 4       | Vue 3 原生路由方案，支持懒加载与守卫                |
| HTTP 请求    | Axios              | 拦截器机制成熟，请求/响应统一处理方便                |
| CSS 预处理   | Sass (SCSS)        | 变量、mixin、嵌套支持，适合设计令牌管理             |
| 代码规范     | ESLint + Prettier + Stylelint | 分别覆盖 JS/TS、格式化与样式，全链路规范      |
| 提交规范     | Commitizen + Husky + lint-staged | 强制提交格式与提交前检查                    |
| 单元测试     | Vitest             | Vite 原生测试方案，与构建配置共享转换管道            |

---

> 本架构文档随项目演进持续更新。新增模块或架构调整时，请同步修订对应章节，保持文档与实现的一致性。
