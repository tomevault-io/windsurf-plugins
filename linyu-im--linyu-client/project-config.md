---
trigger: always_on
description: 林语客户端项目开发规范（技术栈、目录、主题色、国际化、组件化、API、Tauri）
---


# 林语客户端（linyu-client）开发规范

本项目是林语（Linyu）即时通讯桌面客户端，基于 Tauri 2 + Vue 3 + TypeScript 构建。所有代码编写和修改必须遵循以下规范。

## 一、技术栈

### 前端核心
- **框架**: Vue 3.5+（Composition API + `<script setup>`）
- **语言**: TypeScript 6+（strict 模式）
- **构建**: Vite 8+
- **包管理**: pnpm（禁止使用 npm/yarn）

### UI / 样式
- **组件库**: Naive UI 2.44+（自动按需导入，无需手动 import）
- **原子化 CSS**: UnoCSS（preset-wind3，Tailwind 兼容模式）
- **CSS 预处理**: Sass/SCSS（scoped 样式）
- **图标**: SVG sprite（`vite-plugin-svg-icons`，通过 `<svg><use href="#iconName"></use></svg>` 引用）

### 状态管理
- **Pinia 3**（Options API 风格 Store：`state` + `actions`，无 getters）
- **pinia-plugin-persistedstate**：自动持久化
- **pinia-shared-state**：跨 Tauri 窗口状态共享

### 桌面框架
- **Tauri 2**：多窗口架构，Rust 后端
- **HTTP**: `@tauri-apps/plugin-http`（禁止使用 axios）

### 其他
- **路由**: Vue Router 5（HTML5 History 模式）
- **国际化**: vue-i18n 11（中英文双语）
- **工具库**: @vueuse/core、mitt
- **代码质量**: Biome（JS/TS/JSON）+ Prettier（Vue SFC）+ vue-tsc

## 二、代码风格

### 格式化规则
| 规则 | 值 |
|------|-----|
| 引号 | 单引号 |
| 分号 | 不添加（asNeeded） |
| 尾逗号 | 无 |
| 箭头函数参数括号 | 始终添加 |
| Vue SFC 缩进 | `<script>` 和 `<style>` 标签内容缩进 |

### TypeScript 规则
- **strict 模式**：开启所有严格检查
- **noUnusedLocals / noUnusedParameters**：报错
- **允许 `any`**：Biome 中 `noExplicitAny` 已关闭，但应尽量避免
- **JSX**: `jsxImportSource: "vue"`

### 命名规范

#### 文件命名
| 类别 | 风格 | 示例 |
|------|------|------|
| 通用组件 | PascalCase | `LinyuEmpty.vue`、`SvgIconButton.vue` |
| 页面入口 | index.vue | `homeWindow/index.vue` |
| 子页面 | kebab-case | `message.vue`、`contacts.vue` |
| 组合式函数 | useXxx.ts | `useOverflowTooltip.ts` |
| 工具/服务 | camelCase.ts | `http.ts`、`i18n.ts` |
| Store | camelCase.ts | `global.ts`、`user.ts` |
| 类型文件 | camelCase.ts | `auth.ts`、`chat.ts` |

#### 组件命名
- 项目自有组件使用 **`Linyu` 前缀**：`LinyuEmpty`、`LinyuProvider`
- 通用功能组件无前缀：`SvgIconButton`、`ToolBar`
- 模板中一律使用 **PascalCase**：`<ToolBar>`、`<SvgIconButton>`

#### 类型命名
- 对象类型使用 **`interface`**（不使用 `type`，联合类型除外）
- 请求参数以 **`Param`** 后缀：`AccountLoginParam`
- 响应结果以 **`Result`** 后缀：`LoginResult`
- Store state 类型以 **`Store`** 后缀：`GlobalStore`（文件内 `type` 定义，不单独导出）

#### CSS 命名
- SCSS 类名使用 **BEM 简化版**（`block__element`，不使用 `--modifier`）
- CSS 变量命名：`--{类别}-{属性}-{修饰}`，如 `--primary-color`、`--bg-secondary-color`

### 导入规范
- 项目内模块使用 **`@/` 绝对路径**：`import { useUserStore } from '@/stores/user'`
- 禁止使用 `~/` 别名
- Vue 核心 API（`ref`、`computed`、`onMounted` 等）**无需手动 import**（自动导入）
- Naive UI 组件**无需手动 import**（自动注册）
- Naive UI 组合式函数（`useMessage`、`useDialog` 等）**无需手动 import**（自动导入）

## 三、主题色与样式

### 必须使用 CSS 变量，禁止硬编码颜色

主题变量定义在 `src/styles/theme/default.scss`，全局样式在 `src/styles/global.scss`。

```scss
// ✅ 正确
color: var(--primary-color);
background: var(--bg-primary-color);
border: 1px solid var(--border-color);

// ❌ 错误
color: #4C9BFF;
background: #fff;
```

常用变量：`--primary-color`、`--primary-rgb`、`--bg-primary-color`、`--bg-secondary-color`、`--text-color`、`--text-secondary-color`、`--border-color`、`--button-soft-bg`。

### 样式双重体系
- **UnoCSS 原子类**：布局、间距、字体（`flex`、`items-center`、`m-l-10px`、`text-14px`）
- **SCSS scoped 样式**：组件核心结构与主题变量引用
- 主题色通过 CSS 变量：`var(--primary-color)`、`var(--bg-primary-color)`
- UnoCSS 中引用 CSS 变量：`from-[var(--primary-color)]`
- 全局/主题样式放 `src/styles/`，组件私有样式写在 `<style scoped lang="scss">` 内
- Naive UI 主题覆盖在 `LinyuProvider.vue` 中统一配置，勿在业务组件重复定义

### 深色模式
通过 `html[data-theme="dark"]` 切换，样式中需考虑明暗主题时使用 CSS 变量而非固定色值。

## 四、Vue 组件规范

### 必须使用 Composition API + `<script setup>`
```vue
<!-- 正确 -->
<script setup lang="ts">
  const count = ref(0)
</script>

<!-- 错误：禁止使用 Options API -->
<script>
export default { data() { return { count: 0 } } }
</script>
```

### 包含 JSX/TSX 的组件使用 `<script setup lang="tsx">`
```vue
<script setup lang="tsx">
  const renderMessage = () => <span>{t('message.text')}</span>
</script>
```

### 组件通信
- **父子通信**: `defineProps` + `withDefaults` + `defineEmits`
- **跨组件/跨窗口状态共享**: 优先通过 Pinia Store + `pinia-shared-state`（禁止使用 provide/inject）
- **一次性事件通知**: Tauri 事件系统（`emit`/`once`/`listen`）

### 组件化原则
1. **单一职责**：一个组件只做一件事；超过 200 行考虑拆分
2. **可复用逻辑** → `composables/useXxx.ts`
3. **可复用 UI** → `components/` 独立 `.vue`
4. **页面组装**：`views/` 负责布局与数据编排，具体 UI 委托子组件

## 五、目录结构与文件放置

新增文件前**先查阅现有目录**，确认无相同功能目录后再创建。禁止随意新建顶层目录。

```
src/
├── api/            # API 接口层（按业务领域分文件）
├── assets/         # 静态资源（fonts/、icons/）
├── components/     # 公共组件（PascalCase，按功能分子目录）
├── composables/    # 组合式函数（useXxx.ts）
├── constants/      # 常量/枚举定义
├── locales/        # 国际化语言包（zh.json、en.json）
├── router/         # 路由配置
├── services/       # 服务层（i18n 等）
├── stores/         # Pinia 状态管理
├── styles/         # 全局样式 + 主题
├── types/          # TypeScript 类型定义
│   ├── api/        # API 相关类型
│   └── cmd/        # Tauri 命令类型
├── typings/        # 自动生成的类型声明（勿手动修改）
├── utils/          # 工具函数（按功能域分子目录：message/file/desktop/network/screenshot/common）
└── views/          # 页面视图
    └── {windowName}Window/  # 窗口目录（Window 后缀）
        ├── index.vue        # 窗口主页面
        └── pages/           # 子页面（kebab-case）
```

### 新增文件放置规则
- 新 API 接口 → `src/api/` 下新建或追加文件，并更新 `api/index.ts`
- 新类型定义 → `src/types/api/` 或 `src/types/cmd/`
- 新组合式函数 → `src/composables/useXxx.ts`
- 新 Store → `src/stores/`
- 新工具函数 → `src/utils/` 对应子目录（message/file/desktop/network/screenshot/common）
- 新窗口 → `src/views/{windowName}Window/`
- 新子页面 → 对应窗口的 `pages/` 目录
- 新公共组件 → `src/components/`（或功能子目录）
- SVG 图标 → `src/assets/icons/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linyu-im/linyu-client](https://github.com/linyu-im/linyu-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
