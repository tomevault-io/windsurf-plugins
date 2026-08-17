---
trigger: always_on
description: BYOK (Bring Your Own Key) Chrome Extension — 用户插入自己的 API key 获得 AI 浏览器能力。
---

# Chrome AI Agent (Pie)

BYOK (Bring Your Own Key) Chrome Extension — 用户插入自己的 API key 获得 AI 浏览器能力。

## Tech Stack

- Chrome Extension Manifest V3, React 19 + TypeScript 6
- TailwindCSS v4 (Vite plugin, no config file), Vite 8 + @crxjs/vite-plugin 2.4
- pnpm; vitest + happy-dom + @testing-library/react

## Project Structure

- `src/background/` — Service Worker: message routing, port streaming, agent loop dispatch, keep-alive, CDP session lifecycle
- `src/content/` — placeholder (DOM ops 走 `chrome.scripting.executeScript` 注入)
- `src/sidepanel/` — Sidebar UI: Chat (Agent UI) / Settings / SkillsList / SessionDrawer
- `src/lib/model-router/` — Unified LLM interface + tool calling; per-provider modules under `providers/` + two shared cores (`_shared/openai-compat-core.ts`, `_shared/anthropic-sdk-core.ts` 官方 `@anthropic-ai/sdk` 后端) + `registry.ts` 元数据 + id-keyed `providers/index.ts` dispatch（provider 清单见 README）
- `src/lib/dom-actions/` — Self-contained DOM action functions injected via executeScript
- `src/lib/agent/` — ReAct loop, tool registry, prompt builder, token-based context control (compaction / token budget / stale-snapshot elision), `untrusted-wrappers.ts`, `tool-names.ts`(read/write tool 分类)
- `src/lib/agent/tools/` — `keyboard.ts` (CDP) / `skill-meta.ts` (skill CRUD) / `tabs.ts` (cross-tab) / `pdf.ts` (`read_pdf` / `search_pdf` / `get_pdf_outline` tools, all read-class)
- `src/lib/pdf/` — PDF tab detection (`isPdfTab`) + page-range parser (`parsePageRange`)
- `src/offscreen/` — Offscreen document hosting LiteParse v2 WASM (`pdf-parser.html` + `pdf-parser.ts`), in-memory cache, message dispatch（#303 起 MV3 skill 脚本 sandbox iframe 已整体删除——磁盘 skill 脚本只走 daemon CLI，见 `src/lib/skills/`）
- `src/background/offscreen-manager.ts` — Lazy offscreen lifecycle + SW↔offscreen request/response bridge
- `src/lib/skills/` — Skill framework: SkillPackage (frontmatter + virtual file tree) stored in IndexedDB (skill-store), SKILL.md frontmatter parser, builtin packages, getEnabledSkillPackages; skills are accessed via use_skill/read_skill_file mediation tools + a system-prompt catalog and are NOT tools themselves. 磁盘 skill 可捆绑 `scripts/` 下的可执行脚本，经 `run_skill_script` **走 daemon 当作本地 CLI 进程执行**（#303 起：`args`→`process.argv`、结果 print 到 stdout、产物写进 cwd、返回值丢弃；MV3 sandbox / `capabilities.scripts` / `script-decl.ts` 死路径已删）；只有 `scripts/` 下声明过的 entry 可执行，LLM 不能注入代码。**产物按 session 隔离**（#296）：cwd = `~/.pie/sessions/<sessionId>/workspace/`（脚本进程唯一可写区，永不写入任何 skill 目录——含只读副根），daemon run 后扫 workspace（mtime>=startedAt）产出 outputs 清单回 observation（文件名是不可信数据，包 `untrusted_skill_output_list`），LLM 用新 read-class tool `read_skill_output({path})`（→ daemon `read_session_file` RPC，`safeRelPath` 锁死在本 session workspace 内）读回；env 注入 `PIE_SKILL_DIR`（读自身资源）/`PIE_WORKSPACE`；生命周期：`lifecycle.ts` 硬删/归档经 SW `delete-session-workspace` message → daemon RPC 清 workspace，daemon 启动 GC 兜底孤儿（>30 天）。作者文档 `docs/agents/skill-authoring.md`。**daemon 连接且声明 `skill_fs` 时磁盘为唯一真源**（`~/.pie/skills/<name>/`，对齐 Anthropic Agent Skills；`SkillSource` 双后端 + builtin 只读层，mode 判定 `bridgeHasSkillFs()`，panel 走 `skills-action` RPC 单路径）；双根：`~/.agents/skills` 为只读副根（跨 agent 通用目录），daemon skill-store 层合并、主根遮蔽同名、写恒落主根（CoW）、副根删除报 `read_only`；副根 skill 默认关（`filterEnabled` 按 `source` 收窄磁盘默认开），首连经 SkillsList 导入向导多选启用（`agents_import_prompted` 标记）；磁盘 skill 脚本经 daemon srt（@anthropic-ai/sandbox-runtime）默认沙箱执行（写限 workspace/默认断网/敏感读拒），首跑弹信封授权卡（panel-request `skill-grant`，daemon 权威 `SkillAuthPayload`，批准带 `approvedEnvelopeHash` 重调堵 TOCTOU，`grantApproved` 不进 JSON schema——LLM 不能自批）；grant 按能力信封记 daemon 独占 `~/.pie/grants.json`（信封变才重弹），设置页「本地打通」可列出/撤销 + 查最近执行（`list_audit`）。daemon-off 时 builtin/idb skill 无可执行脚本（`run_skill_script` 明确报无脚本）。
- `src/lib/sessions/` — Multi-session persistence: state-machine, lifecycle (archive/delete), pinned-tab-registry, title
- `src/lib/crypto.ts` — AES-GCM encryption helper（与 `src/lib/instances.ts` 配合存 instance API key）
- `src/lib/instances.ts` — Multi-instance CRUD; `instance_${uuid}` + `instances_index` + `active_instance_id`
- `src/lib/migration-v2.ts` — V1→V2 silent migration (`provider_*` → `instance_*`)
- `src/lib/provider-custom-models.ts` — per-provider sticky pool（`pcm_${provider}`）跨 instance 共享自定义 model id
- `src/lib/provider-custom-model-meta.ts` — per-provider sidecar 属性表（`pcmm_${provider}`），给 builtin 自定义模型挂 `vision`/`maxContextTokens`（`tools` 恒 true、不可配）；与 `pcm_${provider}` 的 id 池一一对应，删模型时两边连带清
- `src/lib/openrouter-models-fetch.ts` — `/v1/models` 公共 endpoint normaliser
- `src/types/` — Shared message + agent protocol types

## Commands

- `pnpm dev` — Dev server with HMR
- `pnpm build` — Production build
- `pnpm test` / `pnpm test:watch` — vitest run
- `pnpm typecheck` — `tsc --noEmit`（repo-wide 现已 0 错；任何新报错都是真实回归，必须修，别再当噪音忽略）
- 提交前跑 `pnpm test`、`pnpm typecheck` 与 `pnpm build`（build-time invariants 在 `tool-names.ts`（每个 tool 必须声明 read/write class）/ `tools.ts`（R-iframe-1 write tool 必须 require frameId）会 throw）。注：`tsc` 能跑是靠 tsconfig 的 `ignoreDeprecations: "6.0"`（跨过 `baseUrl` TS5101 硬错）+ `src/global.d.ts` 引用 `chrome`/`vite/client` 类型；移除任一都会让 tsc 退回"哑门禁"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wiseria-ai-labs/pie-ai-agent](https://github.com/wiseria-ai-labs/pie-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
