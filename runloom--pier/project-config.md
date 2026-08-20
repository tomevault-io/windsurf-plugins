---
trigger: always_on
description: 本文件是开发 Pier 时给 Claude Code、Codex 和 OpenCode 共用的项目级上下文（硬约束与治理规则）。
---

# Pier Agent Context

本文件是开发 Pier 时给 Claude Code、Codex 和 OpenCode 共用的项目级上下文（硬约束与治理规则）。  
人类贡献者请从 [`README.md`](README.md) / [`docs/README.md`](docs/README.md) / [`CONTRIBUTING.md`](CONTRIBUTING.md) 进入；不要把本文当作用户手册全文复制进 PR 描述。

## 01 项目定位

Pier 是本地 AI 开发工作台。参考 loomdesk 产品形态，使用 bay 的工具链栈重写。

- 核心能力：稳定终端、dockview panel 布局、代码变更预览、文件查看、多 agent 状态可见性。
- 不做：任务生命周期、SQLite 任务台账、看板、自动调度。
- **核心逻辑优先，拒绝业界能力二次封装**：只实现本产品独有、且依赖 Pier 宿主身份/运行时才能成立的能力；业界已成熟支持的能力（如各 agent 原生 one-shot CLI）直接走原生入口，禁止为「统一抽象 / 便利封装」再造第二套 API 或宿主服务。判定：去掉 Pier 后用户仍能用原生工具完成同一动作 → 不做 Pier 产品封装。
- 持久化分层：用户偏好/布局写 userData JSON；原始终端输出写 transcript 分段文件；代码变更实时读 Git；密钥走 safeStorage。

## 02 技术栈

- Electron 43 · React 19 · TypeScript 6 strict
- electron-vite 5 + Vite 8（main / preload / renderer 三端）
- dockview-react 6.6.1（panel 布局核心：tab + split + floating + drag）
- Tailwind CSS v4 + shadcn primitives
- Zustand 5（client state）
- Biome 2.5 + Ultracite（lint + format 单工具栈）
- pnpm 11
- Vitest 4 + Playwright（测试）

## 03 架构边界

进程边界由 dependency-cruiser 守护：

- `main/` ⊥ `renderer/`（双向禁止）
- `preload/` 只可 import `shared/` + `electron`
- `main/` 内 L1 持久化 ⊥ L2/L3/L4（单向依赖）
- **renderer 业务代码不可直接 import dockview-core/dockview 运行时 API**，必经 `components/workspace/` 边界；panel kit 可使用共享 dockview 类型
- renderer 不同 panel-kits 不跨域 import（走 `components/common` 或 `stores`）
- `src/plugins/builtin/*` 只可 import `src/plugins/api` + `src/shared` + `packages/ui`；宿主只在两个 builtin-catalog 处 import 插件包

### 插件边界是纪律边界，不是安全边界

内置插件与 v1 官方受管理外部插件都属于可信代码：renderer 与宿主同 realm 运行，external main
是普通 Node ESM，可访问 Node 能力。capability 断言（`assertPluginCapability`）、manifest 声明校验、
插件 RPC 的 `pluginId` 作用域和包扫描测试都是工程纪律边界，不构成对恶意代码的防护——main 侧
`authorizeCommand` 当前按 client-kind 授权，不区分插件主体身份。

当前只允许两类插件：

- `src/plugins/builtin/*` 内置插件。
- 官方 bundled / official managed external plugin（例如 `pier.codex`），必须经受管理安装索引、签名官方索引、包校验、不可变版本目录和启动时运行态快照加载。

dev override 只允许开发/测试运行时使用；生产包默认不显示入口、命令返回拒绝结果，即使历史 `index.json` 中已有 dev override 也必须忽略本地路径，并且不得把本地目录标记为官方来源。不得开放第三方插件、任意 registry、任意 git/local 扫描或 marketplace 加载路径。引入第三方插件前必须先设计真正隔离：独立 realm/进程、每插件主体身份、main 侧按插件主体授权、最小权限 host API、供应链签名与回滚策略。

### 宿主弹窗使用规范

宿主级确认/提示弹窗统一走 `src/renderer/components/common/dialogs/host.tsx`：

- 业务代码不要直接 import `@pier/ui/alert-dialog.tsx`；宿主 renderer 使用 `showAppConfirm` / `showAppAlert` / `showAppChoice` / `showAppPrompt`，插件使用 `RendererPluginContext.dialogs` / `ExternalRendererPluginContext.dialogs`。
- builtin 与 external 插件的简单弹窗 API **同构**：`alert` / `confirm` / `choice` / `prompt`；复杂内容另加 `open` / `update` / `close`。
- 布局（路线 B：桌面工具对话框；macOS 优先，全平台同一套壳）：
  - 文案一律左齐；宽度只由 kind 决定，不再切换居中营销卡
  - 密度：`p-5` + `gap-4`、标题 `text-base`、footer **右簇**（禁止 sm 两列等宽铺满）
  - destructive `confirm`：侧标必须用共享 `@pier/ui/status-icon`（与 toast / Alert 同套，`kind="error"`），禁止手写 Lucide 大圆/方底
  - `choice` / 普通 confirm / prompt：**无**侧标；危险只靠按钮色
  - `alert`：单主按钮（右簇）
  - `confirm` / `prompt`：`取消 | 主按钮`（主按钮最右）
  - `choice`：`alt | 取消 | confirm`（例：不保存 | 取消 | 保存）；横排三键
- **`size` 禁止调用方传入**（宿主 `appDialogSizeForKind` / 插件 facade 同构强制）：
  - `alert` / `confirm` / `prompt` → 固定 `sm`
  - `choice` → 固定 `default`（三键横排）
  - 业务与插件 API **不接受** `size` 字段；更长内容走 content dialog（`openAppContentDialog` / `dialogs.open`），不要用宽 confirm 硬塞说明
  - 禁止回退为「每个确认各自传 sm/default」
- `intent`：调用方必填，不要在 `AppDialogHost` 里按标题或文案猜测危险程度
  - 破坏性确认必须显式传 `intent: "destructive"`，普通确认显式传 `intent: "default"`
  - `confirm` / `prompt`：作用在**主按钮**
  - `choice`：作用在 **alt**（不保存/丢弃）；confirm 始终 default 样式
  - 若破坏动作落在 `choice.confirm`（如覆盖），`intent` 仍必须 `"default"`，不能为了“看起来危险”去染 alt
- 取消按钮一律 `outline`（含 destructive 场景）；Esc / 点遮罩 = 取消
- 检查点在 `tests/unit/renderer/notifications/app-dialog-governance.test.ts` 与 `tests/component/app-dialog-host.test.tsx`

复杂内容弹窗（表单、多步、等待态、带自定义 body）统一走宿主 `AppContentDialogHost`：

- 宿主业务使用 `openAppContentDialog` / `updateAppContentDialog` / `closeAppContentDialog`；插件使用 `context.dialogs.open` / `update` / `close`（不要再挂自己的 `@pier/ui/dialog` 产品壳）。
- 插件 renderer 禁止 import `@pier/ui/dialog` 或 `@pier/ui/alert-dialog`；嵌套插件 Dialog（Settings 内再开插件 Dialog）一律禁止。
- **决策树**（必须按此选型，禁止“图省事全走 content dialog”）：
  1. 短成功 / 弱反馈 → toast
  2. 只告知、无决策 → `alert`（固定 `sm`）
  3. 取消 | 确认 → `confirm`（固定 `sm`）
  4. alt | 取消 | 确认 → `choice`（固定 `default`）
  5. 单行输入 + 校验 → `prompt`（固定 `sm`）
  6. 多控件 / 多步 / 等待态 / 结构化结果 → `dialogs.open`（content dialog）
  7. 全页产品壳（设置、物料库）→ 宿主自有 `Dialog`（非插件）
- **无自定义控件的纯确认/提示，禁止塞进 content dialog**（含“title/description + 两个按钮”）。
- 短确认/破坏性确认仍走 `dialogs.confirm` / `showAppConfirm`。
- 模态层级约定：content dialog 栈 > `AppDialogHost` 单槽 > Settings 等宿主产品壳；`AppDialogHost` 新请求会顶替未决简单弹窗，content 栈独立。
- `context.overlays` **已删除**：历史“插件自挂 Dialog 壳”通道不再存在；新代码与存量一律 `dialogs.open`。
- 检查点在 `tests/unit/renderer/plugin-product-dialog-governance.test.ts` 与 content dialog 单测。

#### 弹窗表单规范（交互 + 字段布局，禁止再发明第三套）

弹窗里一旦出现输入控件，只允许下列两种交互模型；壳、footer、字段方向都由模型决定。共享 class 单一来源：`packages/ui/src/dialog-form-layout.ts`（`@pier/ui/dialog-form-layout.ts`）。

| 模型 | 何时用 | 壳 | 字段方向 | Footer | 保存时机 |
|------|--------|----|----------|--------|----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runloom/pier](https://github.com/runloom/pier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
