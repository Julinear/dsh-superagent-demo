# findu 校园失物招领平台

findu 是一个面向高校师生的校园失物招领前端项目，旨在为校园内的失物捡拾与认领提供便捷、高效、可追溯的线上渠道。用户可以发布寻物启事或捡到物品信息，通过分类检索、关键字搜索快速定位物品，并借助站内消息与联系方式完成线下归还。

## ✨ 功能特性

- **失物 / 拾物发布**：支持图文形式发布寻物启事与拾物信息，包含物品名称、分类、丢失 / 捡拾地点与时间等关键字段。
- **分类与搜索**：按物品分类（如证件、电子产品、钥匙、书籍等）筛选，支持关键字模糊搜索。
- **物品详情**：查看物品完整描述、图片、发布人联系方式及拾取 / 丢失地点。
- **个人中心**：管理自己发布的物品、收藏记录与站内消息。
- **消息通知**：物品有新线索时通过站内消息提醒发布人。
- **状态管理**：物品支持「寻找中 / 已找到 / 已归还」状态流转，便于追溯。

## 🛠 技术栈

| 类别       | 选型                                      |
| ---------- | ----------------------------------------- |
| 框架       | Vue 3（Composition API）                  |
| 语言       | TypeScript                                |
| 构建工具   | Vite                                      |
| 状态管理   | Pinia                                     |
| 路由       | Vue Router                                |
| HTTP 请求  | Axios（统一封装请求 / 响应拦截器）        |
| UI 组件库  | 按需引入，遵循项目统一主题                |
| 代码规范   | ESLint + Prettier + Stylelint             |
| 提交规范   | Commitizen + Husky + lint-staged          |

## 🚀 快速开始

### 环境要求

- Node.js >= 18
- pnpm >= 8（推荐使用 pnpm 作为包管理器）

### 安装与运行

```bash
# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev

# 构建生产包
pnpm build

# 本地预览生产构建
pnpm preview
```

## 📁 项目结构

```
findu/
├── public/                # 静态资源（直接拷贝，不经过构建处理）
├── src/
│   ├── api/               # 接口请求，按业务模块拆分
│   ├── assets/            # 需经构建处理的静态资源（图片、字体、样式等）
│   ├── components/        # 全局通用组件
│   ├── composables/       # 组合式函数（可复用逻辑）
│   ├── layouts/           # 布局组件
│   ├── router/            # 路由配置
│   ├── stores/            # Pinia 状态管理
│   ├── styles/            # 全局样式与变量
│   ├── utils/             # 工具函数
│   ├── views/             # 页面级组件，按业务模块组织
│   ├── App.vue
│   └── main.ts
├── .editorconfig
├── .eslintrc.cjs
├── .prettierrc
├── index.html
├── package.json
├── tsconfig.json
└── vite.config.ts
```

## 📐 前端编码规范

### 1. 命名规范

| 类型         | 规范                          | 示例                          |
| ------------ | ----------------------------- | ----------------------------- |
| 目录 / 文件  | kebab-case                    | `lost-detail/`、`use-item.ts` |
| 组件文件     | 大驼峰（PascalCase）          | `ItemCard.vue`                |
| 组件名       | 大驼峰（PascalCase）          | `<ItemCard />`                |
| 变量 / 函数  | 小驼峰（camelCase）           | `getItemList`、`itemCount`    |
| 常量         | 全大写 + 下划线（UPPER_SNAKE）| `MAX_UPLOAD_SIZE`             |
| 类型 / 接口  | 大驼峰（PascalCase）          | `ItemInfo`、`interface User`  |
| CSS 类名     | kebab-case + BEM              | `item-card__title--active`    |
| 事件名       | on + 大驼峰                   | `onItemClick`、`onSubmit`     |

### 2. 组件规范

- **单文件组件**：每个 `.vue` 文件只包含一个组件，文件名与组件名一致。
- **结构顺序**：`<template>` → `<script setup lang="ts">` → `<style scoped>`。
- **Props / Emits**：使用 TypeScript 类型定义，必须声明类型，避免 `any`。
- **组件通信**：父子组件优先通过 `props` / `emits`；跨层级使用 `provide` / `inject` 或 Pinia。
- **通用组件**：放置于 `src/components/`，须具备复用性并提供清晰的 `props` 与事件。
- **页面组件**：放置于 `src/views/`，按业务模块分目录组织。

### 3. TypeScript 规范

- 开启 `strict` 模式，禁止隐式 `any`。
- 公共类型 / 接口统一定义于 `src/types/` 或就近模块的 `types.ts` 中。
- 优先使用 `interface` 描述对象结构，`type` 用于联合类型、工具类型。
- API 响应必须定义对应的类型，接口函数须标注入参与返回类型。

### 4. 样式规范

- 使用 `scoped` 限制样式作用域，避免全局污染。
- 全局变量（颜色、间距、字号）统一定义于 `src/styles/variables.scss`。
- 遵循 BEM 命名规范，避免内联样式。
- 响应式布局优先使用项目定义的断点与栅格系统。

### 5. API 请求规范

- 所有请求统一通过 `src/utils/request.ts` 封装的 Axios 实例发起。
- 接口按业务模块拆分至 `src/api/`，函数名以 `动词 + 资源` 形式命名（如 `getItemList`、`createLostItem`）。
- 统一处理请求 / 响应拦截：携带 token、错误码集中处理。
- 请求参数与返回值均须有类型定义。

### 6. Git 提交规范

采用 [Conventional Commits](https://www.conventionalcommits.org/) 规范：

```
<type>(<scope>): <subject>

<body>

<footer>
```

**type 取值**：

| type     | 说明               |
| -------- | ------------------ |
| feat     | 新功能             |
| fix      | 修复缺陷           |
| docs     | 文档变更           |
| style    | 代码格式（不影响逻辑） |
| refactor | 重构（非新增功能、非修复） |
| perf     | 性能优化           |
| test     | 测试相关           |
| chore    | 构建 / 工具变更    |
| ci       | CI 配置变更        |
| revert   | 回滚提交           |

**示例**：

```
feat(item): 新增失物发布页面的图片上传功能
fix(search): 修复搜索关键字为空时的空指针异常
docs(readme): 补充项目说明与编码规范
```

### 7. 分支与协作规范

- `main`：生产分支，始终保持可发布状态，禁止直接推送。
- `develop`：开发集成分支。
- 功能分支命名：`feat/<模块>-<简述>`，如 `feat/item-publish`。
- 修复分支命名：`fix/<模块>-<简述>`，如 `fix/search-empty`。
- 合并前须通过 ESLint / 类型检查，并通过 Code Review。

### 8. 代码质量保障

- 提交前通过 `lint-staged` 对暂存文件执行 ESLint + Prettier 检查与修复。
- 通过 Husky 在 `pre-commit` 钩子触发检查，`commit-msg` 钩子校验提交信息格式。
- CI 流程执行类型检查、Lint 检查与单元测试，任一失败即阻止合并。

## 📐 架构设计

详细的架构设计说明请参阅 [架构设计文档](./ARCHITECTURE.md)，涵盖整体架构、目录结构、模块职责、数据流、状态管理、路由设计、接口层、组件体系与扩展策略。

## 📋 功能设计

详细的功能设计说明请参阅 [功能设计文档](./FEATURE_DESIGN.md)，涵盖用户认证、物品发布、检索、详情、消息通知与个人中心六大功能域的用户故事、界面交互、状态流转、业务规则、异常处理与验收标准。

## 📄 开源协议

本项目仅用于校园学习与交流，暂未指定开源协议。
