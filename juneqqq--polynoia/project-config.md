---
trigger: always_on
description: > 这份文件是项目级 AI 协作规范。每个新 Claude 会话启动时优先读它。
---

# Polynoia(AgentHub) — Claude 协作规范

> 这份文件是项目级 AI 协作规范。每个新 Claude 会话启动时优先读它。
> Spec:`docs/superpowers/specs/2026-05-23-polynoia-design.md`
> 调研基线:`docs/research/00-SYNTHESIS.md`

> ⚠️ **活跃功能边界(动代码前必看)**:`feature/diff_dev` 正在建「冲突闭环」。**在改 `api/routes.py` 合并/burst 区、`sandbox/_core.py` git helper、pending-edit 轨道、前端 store/PreviewPane/PARTS_REGISTRY 之前**(即使你在做无关任务),先读 [`docs/design/conflict-closed-loop-CHARTER.md`](docs/design/conflict-closed-loop-CHARTER.md) —— 它划清了哪些是共享承重符号(碰了会炸)、哪些可自由动。完整设计见 [`docs/design/conflict-closed-loop-2026-05-30.md`](docs/design/conflict-closed-loop-2026-05-30.md)。

## 1. 项目是什么

**Polynoia**(对外品牌 / 课题代号 AgentHub):IM 形态的多 Agent 协作平台。用户像用 Slack/Lark/微信一样和多个 AI Agent 共处一个对话,Orchestrator 自动拆解任务并并行调度。

详细背景和官方验收映射看 `rule.md`,AI 协作材料索引看 `docs/ai-collaboration.md`。

## 2. 仓库结构

```
polynoia/
├── apps/
│   ├── web/                   Vite + React + TS
│   └── server/                uv + Python FastAPI
├── packages/
│   ├── shared/                跨语言 TS 类型(Pydantic → TS 自动生成)
│   ├── core/                  跨平台业务逻辑(无 DOM/RN)
│   ├── ui-web/                React DOM 组件 — 当前主力
│   └── design-tokens/         跨平台 token
├── docs/
│   ├── research/              已有调研(20 个库源码深读)
│   ├── superpowers/specs/     spec 文档
│   ├── ADR/                   决策记录
│   └── architecture/          图表
├── research/                  调研 clone 归档(1.3GB,只读参考)
├── ui_design/                 设计 handoff(只读)
├── .scratch/                  临时解压目录(gitignore)
├── .skills/                   自定义 skill(add-adapter / add-card-type 等)
└── Makefile                   make dev / test / lint / types / build
```

## 3. 技术栈(锁定)

**后端 apps/server**:
- Python 3.12 + uv(包管理)
- FastAPI + uvicorn + asyncio
- Pydantic v2(domain + IO 模型)
- LiteLLM(custom LLM endpoint)
- SQLite(本地)/ Postgres(P1+)
- Alembic(DB 迁移)

**前端 apps/web**:
- React 18 + TypeScript + **Vite**
- **Radix Primitives**(行为)+ **shadcn/ui**(剥默认样式)
- **Tailwind 4** + CSS variables
- **Motion**(动画)
- **Lucide**(图标)
- **cmdk**(命令面板)
- **dnd-kit**(拖拽)
- **react-hook-form + zod**(表单)
- **@tanstack/virtual**(虚拟列表)
- **CodeMirror 6**(代码编辑,增强:`@codemirror/search` 查找替换 + `@replit/codemirror-vscode-keymap` VSCode 键位 + `@replit/codemirror-minimap` 小地图)+ **`@git-diff-view/react`**(基于 CodeMirror 的 diff 视图)
- **Vercel AI SDK 6**(`ai` 包,作协议层)
- **react-markdown** + rehype(渲染 markdown + mentions)

**显式不引入**:
- ❌ `@assistant-ui/react`(单 user-assistant 模型不适合;借鉴 MessagePart 注册表模式)
- ❌ `@ant-design/x`(设计语言冲突)
- ⏸️ Monaco Editor(**P1+ 推迟,非永久排除**):P0 用增强版 CodeMirror 6(比 Monaco 轻 3–10×,且 Monaco 对我们的文件树/`Ctrl+S→PUT→commit`/diff 评审胶水零支持)。**何时反悔**:真 LSP/IntelliSense 或命令面板成为产品需求 → 届时 power-user opt-in + 懒加载接 `monaco-vscode-api`。详见 [ADR-016](docs/ADR/ADR-016-codemirror-over-monaco.md)

## 4. 核心抽象

### 4.1 数据模型
- ID 全用 ULID
- Message.payload 是 12 种 `kind` 的判别 union:`text / tasks / diff / web / swatches / copy / metrics / sql / schema / logs / api / typing / ask-form`
- Provider → 1:N → Agent(同 provider 可派生多角色 agent)
- Orchestrator **是 Agent**(`role="orchestrator"`),不是特殊代码

### 4.2 MessagePart 注册表
前端核心抽象 — 不是每 message 一个 type,而是 `message.parts: MessagePart[]`。一条消息可含多 parts(text + diff + status 同消息)。组件经注册表分派:

```ts
const PARTS_REGISTRY = { text: TextPart, diff: DiffPart, web: WebPart, ... };
<MessageView msg={msg} parts={PARTS_REGISTRY} />
```

### 4.3 三层协议
- Adapter ↔ Server:PAP(NDJSON stdin/stdout,借 Claude Agent SDK)
- Server ↔ Client:AI SDK 6 UIMessageChunk(SSE/WS,28 chunk types + 自定义 data-${name})
- Client → Server:REST + WS commands

## 5. 常用命令

```bash
# 开发
make dev            # 同时起 server + web(并行,Ctrl-C 全部停)
make server         # 仅 server (uvicorn --reload)
make web            # 仅 web (vite dev)

# 类型同步(后端 Pydantic → 前端 TS)
make types          # 经 datamodel-code-generator 生成 packages/shared

# 测试
make test           # 跑 pytest + vitest
make test-server    # 仅 pytest
make test-web       # 仅 vitest

# 代码质量
make lint           # ruff(server) + biome(web)
make format         # 自动修复

# 数据库
make migrate        # alembic upgrade head
make migration name=add_workspaces  # 新建迁移

# 构建
make build          # 前后端都 build
```

## 6. 关键约束(reading order)

按重要性排序,新会话首次接触代码前必读:

1. `docs/superpowers/specs/2026-05-23-polynoia-design.md` — 完整 spec
2. `docs/research/00-SYNTHESIS.md` — 调研综合(20 个库 + UI 设计)
3. `docs/research/01-ui-design-notes.md` — UI 设计稿解读
4. 然后读你要改的目标代码

### 6.1 编码规则

- **双语(中 / 英)是硬性要求 —— 不是可选项。** 所有**面向用户**的文案**必须**走 i18n 字典 `apps/web/src/lib/i18n.ts` 的 `t(key, lang)`,**禁止在组件里硬编码中文**。覆盖范围:按钮 / 标签 / 菜单项 / `placeholder` / `title` / `aria-label` / tooltip / 空状态 / `toast` · `alert` · `window.confirm` / 表头 / 徽章。
  - 每个 key **必须同时给 `zh` 和 `en`**;组件里用 `const lang = useStore((s) => s.lang)` 取语言。
  - 带变量的文案用 `{name}` 占位,调用处 `t("key", lang).replace("{name}", x)`。
  - **任何新增 / 改动可见文案的改动,必须在同一改动里补齐 en**,并切到 EN 模式自检(右下角语言切换)。
  - 例外(不算用户文案,无需 i18n):用户数据、agent 名 / 头像缩写、示例 / 种子内容、代码注释 / JSDoc / `console.*`。
  - 自检:`grep -rnP '>[^<]*[\x{4e00}-\x{9fff}]' apps/web/src/components` 命中的可见中文(非注释 / 非 `t(`)即为漏网。
- **Pydantic v2 是后端 source-of-truth**;TS 类型经 `make types` 自动生成,**永不手写 packages/shared 内类型**
- **不要在 `packages/core` 内 import `react-dom` / `react-native` / DOM API** — 这层必须跨平台干净
- **每个 Adapter 必须实现 `adapters/base.py:Adapter` Protocol**,把 wire format 翻译成 AdapterEvent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuneQQQ/polynoia](https://github.com/JuneQQQ/polynoia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
