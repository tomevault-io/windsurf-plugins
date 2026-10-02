---
trigger: always_on
description: <!-- AUTONOMY DIRECTIVE — DO NOT REMOVE -->
---

<!-- AUTONOMY DIRECTIVE — DO NOT REMOVE -->
YOU ARE AN AUTONOMOUS CODING AGENT. EXECUTE TASKS TO COMPLETION WITHOUT ASKING FOR PERMISSION.
DO NOT STOP TO ASK "SHOULD I PROCEED?" — PROCEED. DO NOT WAIT FOR CONFIRMATION ON OBVIOUS NEXT STEPS.
IF BLOCKED, TRY AN ALTERNATIVE APPROACH. ONLY ASK WHEN TRULY AMBIGUOUS OR DESTRUCTIVE.
USE CODEX NATIVE SUBAGENTS FOR INDEPENDENT PARALLEL SUBTASKS WHEN THAT IMPROVES THROUGHPUT. THIS IS COMPLEMENTARY TO OMX TEAM MODE.
<!-- END AUTONOMY DIRECTIVE -->

# 注意事项

## 注意遵守的事项

1. 编码过程中注意写注释；
2. 如果 GitHub / NPM 上有成熟的开源方案，直接复用，不要自己实现；
3. 分析 bug 的时候，要从第一性原理出发；
4. 所有实现必须易维护、易扩展，不允许为了当前需求硬编码；
5. 遇到不确定的信息，不要猜测，优先查官方文档或明确指出需要确认的地方；
6. **分支命名规范**：严格采用 `<category>/<kebab-case-description>` 格式：
   - 新功能分支**必须使用完整单词 `feature/`**（⚠️ **严禁使用简写 `feat/`**，避免与 Conventional Commit 的 `feat` type 混淆）；
   - 缺陷修复使用 `fix/` 或 `hotfix/`；
   - 代码重构使用 `refactor/`；
   - 文档更新使用 `docs/`；
   - 性能优化使用 `perf/`；
   - 示例：`feature/desktop-r2-distribution`、`fix/table-cell-editing`、`refactor/split-desktop-view`；
7. **Commit 规范**：严格采用 **Conventional Commits** 规范（格式为 `<type>(<scope>): <subject>`，全小写动词短语）。允许的 type 包括：`feat`, `fix`, `perf`, `refactor`, `docs`, `test`, `chore`, `style`, `ci`, `build`。严禁自由发挥或使用任何 Lore 格式；
8. **Push 前验证规范**：每次执行 `git push` 前，必须在本地依次执行并通过：
   - `pnpm lint`（包含 oxlint、prettier 格式化检查、cargo fmt/clippy）；
   - `pnpm test`（运行所有 package 单元测试，必须 100% 通过；不强制要求 e2e 测试）；
   - `pnpm typecheck`（确保所有 workspace 无 TypeScript 类型错误）；
9. **Push 后 CI 监控规范**：如果当前分支存在关联的 Pull Request，在 `git push` 成功后，必须自动执行 CI 状态监控（如 `gh pr checks` 或 `gh run watch`），观察并向用户汇报 CI 构建与测试结果，确保未引入远程破坏；
10. **严禁无意义的兼容性 Re-export（杜绝代码臃肿）**：当抽取、拆解或新增独立子包/模块时，严禁在旧模块或旧包中为了所谓的“向后兼容”保留无意义的 `re-export` 转发代码。一旦拆出新包，必须直接修改所有历史调用方直接从新包导入，并彻底清理旧模块与无用导出，严防历史包袱累积导致代码库日益臃肿；
11. **PR 合并与分支清理规范**：使用 GitHub CLI（`gh pr merge`）执行 PR 合并时，必须携带 `--delete-branch`（或 `-d`）参数，合并后自动清理远程分支与本地分支，避免已合入分支长期累积堆积；严禁删除 `main`、`dev`、`beta` 保护分支。

## 架构边界设计

设计新能力或修复跨层问题时，优先把稳定能力收敛到拥有该职责的核心模块中，外部接入层只做数据获取、协议适配和结果注入。不要让外部 provider、AI 请求层、平台适配层或临时 UI 逻辑直接承担编辑器语义、交互状态、渲染规则或验收逻辑。

具体要求：

1. 先判断问题的本体属于哪个领域：编辑器交互、文档模型、文件系统、AI provider、平台 runtime、主题样式等；
2. 领域内的核心模块负责提供可复用能力和行为契约，接入层通过明确 API 注入输入，不复制核心行为；
3. AI 相关能力中，AI 层只负责和模型交互、解析推理结果；编辑器层负责 suggestion 的展示、接受、取消、失效、选区和换行等交互语义；
4. 验证要按边界拆开：provider/request parsing 单测独立于编辑器行为测试，编辑器能力测试不依赖真实模型或网络；
5. 如果发现某个修复需要在调用方硬算核心领域状态，应优先回到核心模块设计预留能力。

更详细的边界设计说明见 [架构能力边界设计原则](./docs/agent/architecture/capability_boundary_design_principles.md)。

## 文档规范

你可以**按需阅读** [agent/index](./docs/agent/index.md) 这篇文档，来获得相应的信息，**注意是按需阅读目录下的文件**，不可一下加载所有的文档。

如果你有文档需要记录，只可以记录在 `./docs/agent/` 目录下，并且更新 `./docs/agent/index.md` 这篇目录文档，在记录时说清楚文档的用途，以便将来更好的查询。

<!-- omx:generated:agents-md -->

# oh-my-codex - Intelligent Multi-Agent Orchestration

You are running with oh-my-codex (OMX), a coordination layer for Codex CLI.
This AGENTS.md is the top-level operating contract for the workspace.
Role prompts under `prompts/*.md` are narrower execution surfaces. They must follow this file, not override it.
When OMX is installed, load prompts from `./.codex/prompts`, skills from `./.agents/skills`, and native agents from `./.codex/agents`.

<guidance_schema_contract>
Canonical guidance schema for this template is embedded in this `AGENTS.md` contract.

Required schema sections and this template's mapping:

- **Role & Intent**: title + opening paragraphs.
- **Operating Principles**: `<operating_principles>`.
- **Execution Protocol**: delegation/model routing/agent catalog/skills/team pipeline sections.
- **Constraints & Safety**: keyword detection, cancellation, and state-management rules.
- **Verification & Completion**: `<verification>` + continuation checks in `<execution_protocols>`.
- **Recovery & Lifecycle Overlays**: runtime/team overlays are appended by marker-bounded runtime hooks.

Keep runtime marker contracts stable and non-destructive when overlays are applied:

- `<!-- OMX:RUNTIME:START --> ... <!-- OMX:RUNTIME:END -->`
- `<!-- OMX:TEAM:WORKER:START --> ... <!-- OMX:TEAM:WORKER:END -->`
</guidance_schema_contract>

The workspace now uses pnpm workspaces with a Tauri + React desktop app. Useful commands:

- `find docs -type f -name '*.md'`: list documentation files.
- `rg "Milkdown|MDX|Tauri" docs/`: search project decisions.
- `git status`: check pending changes once a Git repository is initialized.
- `pnpm install`: install workspace dependencies.
- `pnpm dev`: start the desktop web shell through Vite.
- `pnpm tauri dev`: run the Tauri desktop app.
- `pnpm typecheck`: run TypeScript checks for every workspace.
- `pnpm test`: run Vitest suites.
- `pnpm build`: build all workspaces.

<operating_principles>

- Solve the task directly when you can do so safely and well.
- Delegate only when it materially improves quality, speed, or correctness.
- Keep progress short, concrete, and useful.
- Prefer evidence over assumption; verify before claiming completion.
- Use the lightest path that preserves quality: direct action, MCP, then delegation.
- Check official documentation before implementing with unfamiliar SDKs, frameworks, or APIs.
- Within a single Codex session or team pane, use Codex native subagents for independent, bounded parallel subtasks when that improves throughput.
<!-- OMX:GUIDANCE:OPERATING:START -->

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wmasfoe/md-editor](https://github.com/wmasfoe/md-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
