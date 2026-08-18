---
trigger: always_on
description: > 作用域:`g:\IHUI-AI` 仓库根目录及所有子目录。
---

# AGENTS.md — IHUI-AI 项目 Agent 指南

> 作用域:`g:\IHUI-AI` 仓库根目录及所有子目录。
> 历史案例归档见 `.trae-cn/archive/AGENTS_history.md`。
> 本文件为精简版(2026-07-25 重构,原 783 行 → ≤400 行),保留所有强制规则核心条款。

---

## 1. 任务计划文档规则(强制)

- 项目**唯一**任务计划文档是 `PROJECT_PLAN.md`(根目录),所有任务计划、进度更新、待办清单、状态变更**只写**此文件,**不得**在 `.trae/`、`docs/`、根目录或其他位置新建计划/TODO/ROADMAP 文件。
- 完成任务后 `[ ]` → `[x] ✅(日期)`;新增任务追加到对应优先级(P0/P1/P2)末尾。commit message:`feat`/`fix`/`docs`/`chore`/`test`/`refactor` 前缀。

### 归档机制

- 已完成任务条目(`### XXX(已完成 ✅ ...)` 标题)**禁止直接删除**,必须两步走:① 把完整任务条目(标题 + 正文)移动到 `.trae-cn/archive/PROJECT_PLAN_YYYY-MM-DD.md`;② 在 `PROJECT_PLAN.md` 原位置留 HTML 注释占位:`<!-- 已归档(YYYY-MM-DD):XXX 任务,完整内容在 .trae-cn/archive/PROJECT_PLAN_*.md -->`。
- **自动归档**:`scripts/archive-completed-tasks.mjs` 扫描完成 ≥7 天的条目,post-commit 钩子自动 `--auto-commit`,归档 commit 设 `IHUI_ARCHIVE_COMMIT=1` 防递归。
- **手动触发**:`pnpm archive` / `--all`(全部)/ `--days 3`(自定义)/ `--dry-run`(预览);跳过用 `HUSKY_SKIP_ARCHIVE=1 git commit`。
- **守门**:`scripts/check-project-plan-archive.mjs` + pre-commit 第 13c 项。历史案例见 `.trae-cn/archive/AGENTS_history.md`。

### 唯一例外

- `/goal` 模式:`.trae-cn/goal-runtime/STATE.md` + `loop-run-log.md`(临时,目标结束后删除);skills:`.trae-cn/skills/SKILL.md`(AI 工具配置,非计划文档)。

---

## 2. 项目概览

IHUI-AI 是全栈 AI 平台(TS Monorepo + pnpm workspace + Turborepo),8 端清单:

- `apps/api`(Fastify 5 + Drizzle ORM 0.38 + PostgreSQL)
- `apps/web`(Next.js 16.2.12 + React 19 + Tailwind 4 + shadcn/ui)
- `apps/ai-service`(FastAPI + LangGraph + LiteLLM + MCP)
- `apps/miniapp-taro`(Taro 4 + React)
- `apps/desktop` / `apps/extension` / `apps/mobile-rn` / `apps/cli`(各端独立)
- `packages/`(database / auth / types / ui / config / eslint-config / tsconfig)

---

## 3. 代码风格

- 做减法,最小化代码,零冗余。复用现有代码和模式,不创建文档文件(除非明确要求),不加 copyright/license header。

### TypeScript 类型零技术债(强制)

- **尽最大程度禁用 `any`**,优先用 `unknown` + 类型守卫 / `as const` / 泛型 / 条件类型 / 工具类型(`Pick`/`Omit`/`Record`/`Partial`/`Required`/`ReturnType`/`Parameters`)/ 精确接口替代,禁止把 `any` 当"类型兜底"逃避设计。
- **深度分析 TS 用法**:函数签名(入参/出参/泛型约束)、对象字段、API 响应、props、state 必须显式标注精确类型;能由 `tsc` 推断且可读性良好的局部变量可省略,但禁止"省略 = 不写类型"扩散到公共 API。
- **必须用 `any` 的例外(三选一)**:① 第三方库无 `@types` 或类型声明缺失;② 泛型推断失败且无法用 `unknown` + 守卫替代;③ 跨包循环依赖无法用类型导入断言解决。**必须**附行内注释 `// FIXME(any): 原因 + 移除计划 + 截止版本`,后续 PR 必须清理。
- **不留技术债**:新代码 `tsc --noEmit` 必须 0 错误,禁止"先 any 后修"占位;重构遗留 `any` 必须替换为精确类型,**禁止**复制粘贴扩散 `any`;PR 引入新的 `any` 必须在 PR 描述说明例外依据。
- **守门**(分层渐进):
  - **过渡期(当前生效)**:`@typescript-eslint/no-explicit-any: error`(packages/eslint-config/index.js,syntax-level 不需 type info,已生效)。lint-staged 对 staged 源码文件触发阻塞,新代码引入 `any` 会在 `pnpm lint` 报错。
  - **目标态(已评估,推迟启用)**:`@typescript-eslint/no-unsafe-assignment` / `no-unsafe-member-access` / `no-unsafe-call` / `no-unsafe-return` / `no-unsafe-argument` 五条规则设为 `error`。**2026-07-28 评估结论**:实测启用 `recommendedTypeChecked` + `projectService` 后,(a) 性能不可接受 — cli 包(最小)lint 时间 2.7s -> 51.9s(慢 19 倍),web/api 包预计 30s -> 600s+;(b) 历史错误多 — cli 包已报 30+ 处(`require-await` / `no-unsafe-assignment` / `no-base-to-string` / `restrict-template-expressions` / `no-unnecessary-type-assertion`),全量预计 300+ 处;(c) 配置陷阱 — `eslint.config.js` 不被 tsconfig include 需 `allowDefaultProject`,JS 文件需单独豁免 `no-unsafe-*`。**启用前置条件(全部满足才可启用)**:1. lint 性能优化方案落地(eslint cache 持久化 / 仅 staged 文件 typed-lint / CI 才跑全量 typed-lint);2. 历史类型错误清零(`require-await` 等非 unsafe 错误先修);3. `allowDefaultProject` 配置就绪。
  - **测试文件豁免**:`**/*.test.ts` / `**/*.spec.ts` / `**/tests/**` / `**/test/**` / `**/e2e/**` 路径下的 mock/stub 代码允许 `any`(mock 类型断言必需),启用 typed-linting 后通过 `files` overrides 关闭上述规则。
  - CI `pnpm typecheck` 全绿方可合并。

### 共享层优先(强制)

- **写新代码前必须先查共享层**,确认是否已有现成实现。禁止在端内(apps/*)重新实现 `packages/` 已提供的功能。
- **检查清单(按顺序)**:
  1. **hooks**: `packages/shared/src/hooks/` — 基础 hook(clipboard/debounce/countdown/form/mounted/pagination 等)和业务 hook(auth/chat/agents/articles/agent-runtime/confirm-dialog 等)共 16 个。各端 `hooks/` 目录应只做 re-export wrapper + 平台 adapter,不得独立实现。
  2. **utils**: `packages/shared/src/utils/` — 工具函数(date-utils/dangerous-command-detector/format/file-helpers/error-messages/jwt-utils/ai-skill-variables 等)。各端 `lib/` 目录应只做 re-export wrapper。
  3. **types**: `packages/types/src/` — 所有跨端类型(ChatMessage/MessageInputFile/WorkspacePermissionMode/User/ApiRequest 等)。禁止在端内重新声明同名类型。
  4. **api-client**: `packages/api-client/src/` — 所有 API 调用。禁止在端内直接用 `fetch`/`axios`/`Taro.request` 调后端,必须走 `@ihui/api-client`。端内可保留 re-export + 平台 adapter(如 AsyncStorage 持久化)。
  5. **stores**: `packages/shared/src/stores/` — 共享 store 工厂(createAuthStore/createThemeStore)。各端 store 应调工厂 + 注入平台 transport,不得重新定义 state shape。
  6. **constants**: `packages/shared/src/constants/` — 跨端常量(storage key/URL/locale key 等)。禁止在端内硬编码同名常量。
- **如果共享层没有**:
  - 评估是否跨端可用 → 如果是,先提取到 `packages/shared/` 再在端内 import,不得直接在端内写。
  - 如果确认平台特有(依赖 DOM/RN API/Taro API)→ 可在端内实现,但必须在文件头注释说明 `// 平台特有:依赖 [DOM/RN/Taro] API,不适合共享`。
- **工厂模式优先**:跨端 hook/util 用工厂函数 + 依赖注入(参考 `createUseClipboard` / `createAuthStore`),各端传入平台 adapter。不得用 `if (Platform.OS === 'web')` 条件分支在共享层处理平台差异。
- **守门**:PR review 时检查是否有端内文件重新实现了共享层已有功能。发现重复 → 要求改为 import 共享层。

---

## 4. 前端 UI 约束

- compact 紧凑、elegant 优雅。hover 用 subtle 颜色变化,**不要蓝色发光边框**。复用 `packages/ui-react` 的 Card/Button/Input/Dialog。每个页面 < 250 行。时间用 `Intl.DateTimeFormat`,头像用 initials。状态徽章:draft 灰 / published 绿。积分正数绿色,负数红色。

### 圆角守门(强制)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IHUI-INF-AI/IHUI-AI](https://github.com/IHUI-INF-AI/IHUI-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
