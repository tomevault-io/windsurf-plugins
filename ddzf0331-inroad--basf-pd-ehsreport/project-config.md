---
trigger: always_on
description: 本文件是本项目后续 **PC Web 管理端** 与 **移动端 H5 / WebView** 开发的统一约束。执行任何前端任务前，先判断目标端，再应用“通用规范 + 对应端专属规范”。
---

# 前端开发 Agent 规范

本文件是本项目后续 **PC Web 管理端** 与 **移动端 H5 / WebView** 开发的统一约束。执行任何前端任务前，先判断目标端，再应用“通用规范 + 对应端专属规范”。

## 0. 规范层级与详细参考

本文件负责目标端判定、通用约束和必须遵守的核心规则。两份 v1.0 文件是详细设计与实现参考，仅在本文件没有覆盖或描述不够具体时读取：

| 目标端 | 详细参考文件 | 适用内容 |
|--------|--------------|----------|
| Web | `docs/standards/前端Agent模板-Vue版-v1.0.md` | Ant Design Vue、PC 布局、字体与间距、组件样式、页面模式及实现示例 |
| H5 | `docs/standards/前端Agent模板-H5版-v1.0.md` | Vant、移动端适配、安全区、触控、字体与间距、组件样式及实现示例 |

### 0.1 读取规则

1. 始终先读本文件并按第 1 节确定目标端。
2. 本文件已经明确的规则直接执行，不需要用详细参考文件重新解释。
3. 遇到主题、字体、间距、组件状态、页面布局、交互细节或代码落地方式不够明确时，只读取对应端详细参考文件中的相关章节。
4. Web 任务不得引用 H5 参考文件，H5 任务不得引用 Web 参考文件。
5. 两份详细文件是通用模板，其中的占位符、示例项目名、示例模块、示例路由和初始化命令不是本项目的既定事实，不得直接照搬。
6. 详细参考与本文件冲突时以本文件为准；详细参考与用户当前明确要求冲突时，以用户当前要求为准并说明影响。
7. 若详细参考仍不能唯一确定实现方案，列出歧义并向用户确认，禁止自行选择。

规则优先级：**用户当前明确要求 > 本 `AGENTS.md` > 对应端 v1.0 详细参考 > 其他通用建议**。

## 1. 先判断目标端

### 1.1 判定顺序

1. 用户明确说“Web、PC、后台、管理端、桌面端”时，执行 **Web 规范**。
2. 用户明确说“H5、移动端、手机端、WebView”时，执行 **H5 规范**。
3. 用户未明确时，根据正在修改的应用目录、依赖和现有组件判断：
   - `ant-design-vue` 属于 Web；
   - `vant` 属于 H5；
   - 已有页面使用哪个端的布局和组件，就沿用该端规范。
4. 若仍无法唯一判断，或需求描述与现有技术栈冲突，**停止编码并向用户确认**，不得自行猜测。

### 1.2 严格隔离

| 目标端 | UI 组件库 | 页面骨架 | 适配重点 |
|--------|------------|----------|----------|
| Web | Ant Design Vue 4 | 深色侧栏 + 白色顶栏 + 内容区 | 桌面端信息密度、表格与表单 |
| H5 | Vant 4 | NavBar + 内容区 + TabBar | 375 设计稿、触控、安全区 |

- Web 与 H5 规范不可混用，同一个应用不得同时引入 Ant Design Vue 与 Vant。
- 不能仅凭路由名或页面名称猜目标端。
- 当前仓库尚未确定单应用或多应用目录结构。首次创建应用时，若用户未指定应用类型和目录，必须先确认；不得擅自创建 `src/`、`apps/web/` 或 `apps/h5/`。

## 2. 未确认的项目配置

以下信息未确认时必须询问用户，禁止编造：

- 项目正式名称与描述；
- Web、H5 各自的源码目录和开发端口；
- API Base URL、API Prefix；
- Node.js 版本；
- 业务模块、路由、权限和优先级；
- H5 设计稿宽度（未另行确认时，附件规范建议值为 375，但开始实现前仍需结合现有项目配置核对）。

固定视觉约定：主色 `#15C5C5`；成功 `#03B361`；警告 `#FDC319`；错误 `#E43C4C`；信息 `#3492F8`。颜色统一写入主题配置与全局样式，禁止散落在业务代码中硬编码。

## 3. 通用开发规范

### 3.1 技术基础

- Vue 3、TypeScript、Pinia、Vue Router 4、vue-i18n、Vite+、axios、Sass、ECharts、dayjs。
- 使用 Oxc / Vite+：`vp fmt`、`vp check`；禁止新增 ESLint 或 Prettier。
- 依赖使用 npm 最新稳定版并锁定精确版本，提交 lock 文件；禁止 beta、rc、canary 等预发布版本。
- `vite.config.ts` 从 `vite-plus` 导入 `defineConfig`，配置 `@` 指向 `src`、开发端口以及 `fmt` / `lint`。

### 3.2 Agent 执行流程

1. 阅读本文件、现有代码、项目脚本和同类页面；布局或 UX 任务在可用时先读取 `ui-ux-pro-max` Skill。若该 Skill 不可用，明确告知用户并严格依据本文件和现有设计实现。
2. 按第 1 节确定目标端；不明确就先询问。
3. 搜索可复用组件、API、composable 和 store，优先使用目标端 UI 组件库。
4. 定义可验证的验收标准，只做需求必要的最小改动，不改无关文件。
5. 新页面同步处理路由、菜单或 Tab、双语 locale；仅在跨页面共享状态时新增 store。
6. 修改 `src/` 后运行 `vp check`，再运行 `vp build`；失败时修复并重新验证直至通过。仅修改文档时不要求构建。
7. UI 改动必须按目标端视口进行视觉检查；H5 还需使用移动设备模式或真机检查。
8. 交付时说明完成内容、改动文件和实际验证结果。

### 3.3 最小改动与禁止事项

- 不修改、重构、格式化或删除与需求无关的代码。
- 不为一次性需求增加抽象层、扩展点或额外配置。
- 组件使用 PascalCase `.vue`；composable 使用 `use` 前缀；API 和 store 使用 camelCase。
- 禁止使用 `any`；确有例外必须在代码旁说明原因。
- 单文件超过 300 行时，按已有代码风格拆分职责，不能通过压缩可读性规避限制。
- 页面组件内禁止直接调用 axios；请求统一封装在 `src/api/`，底层走 `src/utils/request.ts`。
- UI 文案必须通过 `vue-i18n` 的 `t()` 获取，禁止硬编码用户可见文案。
- 不重复实现 UI 组件库已有组件。
- 业务逻辑下沉到 composable；仅跨页面共享状态进入 Pinia store。
- 禁止前端直连 Supabase 写库；数据流为 `src/api/*` → 后端 API → Supabase。

### 3.4 推荐目录与新增模块清单

在已有项目中必须沿用实际目录结构，不得为贴合本节而重构。全新单应用可参考：

```text
src/
├── api/
├── assets/styles/
├── components/common/
├── components/business/
├── composables/
├── config/
├── constants/
├── layouts/BasicLayout.vue
├── locales/zh-CN/
├── locales/en-US/
├── plugins/i18n.ts
├── router/
├── stores/
├── types/
├── utils/request.ts
├── views/
├── App.vue
└── main.ts
```

新增模块时核对：

```text
src/views/<模块>/
src/api/<模块>.ts
src/router/index.ts
src/locales/zh-CN/<模块>.ts
src/locales/en-US/<模块>.ts
src/types/<模块>.ts        # 仅需要共享类型时
src/stores/<模块>.ts       # 仅跨页面共享状态时
```

### 3.5 路由、国际化与主题

- 路由懒加载；`meta` 至少按需包含 `titleKey`、`requiresAuth`、`permission`。
- i18n key 使用 `模块.场景.含义`，如 `common.action.save`。
- 新页面同时维护 `zh-CN` 与 `en-US`；业务数据本身不翻译。
- 语言切换通过 `stores/app.ts` 联动 vue-i18n、目标 UI 组件库和 dayjs，并以 `app-locale` 写入 `localStorage`。
- 默认语言为 `zh-CN`，同时支持 `en-US`。
- 主题颜色集中于 `src/config/theme.ts` 和全局样式；禁止业务组件硬编码主题色。

### 3.6 API、鉴权与错误处理

```typescript
interface ApiResponse<T> {
  code: number
  data: T
  message: string
}

interface PageResult<T> {
  list: T[]
  total: number
}
```

- `request.ts` 从 `VITE_*` 环境变量读取 API 配置，不在页面中硬编码地址。
- 401 清理登录状态并退出；403 使用目标端反馈组件提示无权限；5xx 统一提示。
- 鉴权由请求拦截器、`router.beforeEach` 与 RBAC 共同处理。
- 如需种子 Mock，仅使用后端 `POST <API_PREFIX>/seed`，禁止在前端模拟直连数据库写入。

### 3.7 通用嵌入模式

- iframe、微前端或 WebView 嵌入时，只渲染 `<RouterView />`，隐藏所属端的导航壳层和语言切换。
- 统一在 `src/composables/useEmbedMode.ts` 判定，并由 `BasicLayout.vue` 条件渲染；禁止每个页面重复判断。
- 通用判定：`?embed=1`、`?embedded=true` 或 `window.self !== window.top`；H5 还可按原生约定支持 `?webview=1`。

## 4. Web 专属规范（PC 管理端）

仅当目标端判定为 Web 时应用本节。

### 4.1 Web 技术与关键路径

- UI：Ant Design Vue 4，优先级为 Ant Design Vue → `components/common/` → 自写。
- 图标：`@ant-design/icons-vue`。
- 关键路径：
  - 入口：`src/main.ts`
  - 主题：`src/config/theme.ts`、`src/App.vue`
  - 布局：`src/layouts/BasicLayout.vue`
  - 路由：`src/router/index.ts`
  - i18n：`src/locales/`、`src/plugins/i18n.ts`、`antdLocale.ts`
  - API：`src/api/`、`src/utils/request.ts`
- 禁止引入或使用 Vant、Element Plus。

### 4.2 Web 系统布局

全新 Web 应用初始化时必须实现以下 `BasicLayout`：

```text
┌─ SideBar ─────┬─ Header（白底 64px）────────────────────┐
│ 深色 256px    ├─ Content（灰底、内边距 24px）───────────┤
│ Logo + 菜单   │  <RouterView />                         │
└───────────────┴─────────────────────────────────────────┘
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddzf0331-inroad/basf-pd-ehsreport](https://github.com/ddzf0331-inroad/basf-pd-ehsreport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
