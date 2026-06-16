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
- `src/lib/agent/` — ReAct loop, tool registry, prompt builder, sliding window, `untrusted-wrappers.ts`, `tool-names.ts`(read/write tool 分类)
- `src/lib/agent/tools/` — `keyboard.ts` (CDP) / `skill-meta.ts` (skill CRUD) / `tabs.ts` (cross-tab) / `pdf.ts` (`read_pdf` / `search_pdf` / `get_pdf_outline` tools, all read-class)
- `src/lib/pdf/` — PDF tab detection (`isPdfTab`) + page-range parser (`parsePageRange`)
- `src/offscreen/` — Offscreen document hosting LiteParse v2 WASM (`pdf-parser.html` + `pdf-parser.ts`), in-memory cache, message dispatch
- `src/background/offscreen-manager.ts` — Lazy offscreen lifecycle + SW↔offscreen request/response bridge
- `src/lib/skills/` — Skill framework: SkillPackage (frontmatter + virtual file tree) stored in IndexedDB (skill-store), SKILL.md frontmatter parser, builtin packages, getEnabledSkillPackages; skills are accessed via use_skill/read_skill_file mediation tools + a system-prompt catalog and are NOT tools themselves.
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
- 远端 GH 操作前先 `gh auth switch --user WiseriaAI`；默认 active 账号 `wenkang-xie` 在 org 仓库无 admin scope（Pages API / repo settings 会 404）

## Development

1. `pnpm dev` 启 Vite dev server
2. `chrome://extensions` 开启 Developer mode
3. Load unpacked 加载 `dist/` 目录（指向**主仓库**的 `dist/`，一次指定后只认这个路径）
4. 点击扩展图标打开 side panel

## 本地开发约定（worktree 默认 + dist 同步）

两条默认规则，无需每次提醒：

1. **开发新功能默认起一个 worktree（不是在主检出上开新分支）。** 用 `superpowers:using-git-worktrees` / 原生 `EnterWorktree`，CC 原生 worktree 落在 `.claude/worktrees/<name>/`。主仓库检出始终留在 `main`，供 Chrome 加载 dist。文档/小 chore 可直接在 main 上做，不强制 worktree。
2. **用户说「需要测试 / 测一下」时，自动把 worktree 的构建产物同步到主仓库 dist——不必等用户再提醒。** 流程：在 worktree 里 `pnpm build`，再 `pnpm sync:dist`（= `scripts/sync-dist.sh`），然后让用户去 `chrome://extensions` 点刷新即可测到新代码。`dist/` 已 gitignore，灌进主仓库不污染 git。脚本用 git 自发现「当前 worktree → 主仓库」两端路径，从任意 worktree 子目录运行都对；已在主仓库时自动跳过。
   - 为何不是 hook：`UserPromptSubmit` hook 会在「用户发话、尚未 build」时触发，复制到的是旧/空 dist（时序错）；`PostToolUse` on build 又会被提交前的例行 `pnpm build` 过度触发。正确做法是 build 之后按口令执行同步，因此落在约定+脚本，而非 hook。

## Release

`.github/workflows/release.yml` 是唯一发布入口。**不要**手动 `gh release upload` 传 zip——除非是已发布 tag 的紧急补救。

发新版流程：
1. bump `package.json` 和 `manifest.json` 的 `version`（必须一致），commit
2. `git tag v0.x.y && git push origin v0.x.y`
3. 在 GitHub 上 publish release notes（tag 已存在即可）
4. tag push 触发 workflow → CI 跑 `pnpm build` → 验 manifest invariant → 打包 `pie-0.x.y.zip` → 上传到对应 release

Workflow 内置 invariant（任一失败则 CI fail，不会上传）：
- `dist/manifest.json` 的 `background.service_worker` 和 `content_scripts[0].js[0]` 必须以 `.js` 结尾（不是 `.ts`）
- `manifest.version` 必须等于 tag 去掉 `v` 前缀（即 package.json / manifest.json 没 bump 就发 tag 会被拦下）

补传历史 tag：`gh workflow run release.yml -f tag=v0.x.y`（用 `workflow_dispatch`，会 checkout 那个 tag commit 重 build + `--clobber` 覆盖）。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WiseriaAI/pie-ai-agent](https://github.com/WiseriaAI/pie-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
