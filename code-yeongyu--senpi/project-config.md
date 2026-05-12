---
trigger: always_on
description: **Generated:** 2026-05-11 · **Commit:** 4b3f407d · **Branch:** main
---

# senpi-mono

**Generated:** 2026-05-11 · **Commit:** 4b3f407d · **Branch:** main

## OVERVIEW

Opinionated fork of [badlogic/pi-mono](https://github.com/badlogic/pi-mono). TypeScript monorepo (npm workspaces, also bun + pnpm verified). Coding agent CLI is rebranded **senpi**. Extension-first philosophy: prefer extension over core source change; when core change is unavoidable, document it in a `changes.md` next to the modified files.

## STRUCTURE

```
senpi-mono/
├── packages/
│   ├── ai/             # Multi-provider LLM API (@earendil-works/pi-ai)
│   ├── agent/          # Agent runtime + harness (@earendil-works/pi-agent-core)
│   ├── coding-agent/   # senpi CLI (@code-yeongyu/senpi) — primary fork target
│   ├── tui/            # Differential renderer (@earendil-works/pi-tui)
│   ├── web-ui/         # Lit chat components (@earendil-works/pi-web-ui)
│   ├── mom/            # Empty stub (only dist/, tsconfig.json) — do not touch
│   └── pods/           # Empty stub (only dist/) — do not touch
├── scripts/            # build-all.mjs, release.mjs, verify-package-managers.mjs, etc.
├── .github/workflows/  # ci.yml, build-binaries.yml, pr-gate.yml, openclaw-gate.yml, issue-gate.yml, approve-contributor.yml
├── .husky/pre-commit   # npm run check (Biome+tsgo) + conditional verify:pms + browser-smoke
├── biome.json          # TAB indent (width 3), 120 line width
├── tsconfig.json       # noEmit root config + workspace path aliases
├── tsconfig.base.json  # ES2022/Node16, strict, decorators on
├── pi-test.sh          # Live-API integration runner (env-gated)
└── local-ignore/       # gitignored — DO NOT regenerate AGENTS.md inside
```

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Add LLM provider | [`packages/ai/src/providers/`](packages/ai/src/providers/AGENTS.md) — 7-step checklist |
| Add text-format tool-call protocol (Hermes/XML/YAML) | [`packages/ai/src/tool-call-middleware/`](packages/ai/src/tool-call-middleware/AGENTS.md) |
| Add senpi tool/command/flag | builtin extension in [`packages/coding-agent/src/core/extensions/builtin/`](packages/coding-agent/src/core/extensions/AGENTS.md) |
| Add core senpi tool | [`packages/coding-agent/src/core/tools/`](packages/coding-agent/src/core/tools/AGENTS.md) — only if upstream parity required |
| Change senpi system prompt | [`packages/coding-agent/src/core/dynamic-prompt/`](packages/coding-agent/src/core/dynamic-prompt/AGENTS.md) (or extensions/builtin/prompt-preset/ for per-model) |
| Modify built-in extensions | [`packages/coding-agent/src/core/extensions/builtin/`](packages/coding-agent/src/core/extensions/builtin/AGENTS.md) — see per-extension AGENTS.md |
| Sub-agent / background-task work | [`packages/coding-agent/src/core/extensions/builtin/background-task/`](packages/coding-agent/src/core/extensions/builtin/background-task/AGENTS.md) |
| Permission system / external agent profiles | [`permission-system/`](packages/coding-agent/src/core/extensions/builtin/permission-system/AGENTS.md); agent profiles live in sibling repo `../pi-extensions/pi-agent-system` |
| Compaction policy | [`compaction/`](packages/coding-agent/src/core/extensions/builtin/compaction/AGENTS.md) (builtin extension; `core/compaction/` only holds constants) |
| Per-model prompt presets | [`prompt-preset/`](packages/coding-agent/src/core/extensions/builtin/prompt-preset/AGENTS.md) |
| GPT `apply_patch` tool | [`gpt-apply-patch/`](packages/coding-agent/src/core/extensions/builtin/gpt-apply-patch/AGENTS.md) |
| Add TUI component | [`packages/coding-agent/src/modes/interactive/`](packages/coding-agent/src/modes/interactive/AGENTS.md) |
| Cross-cutting utility (git, shell, paths, image) | [`packages/coding-agent/src/utils/`](packages/coding-agent/src/utils/AGENTS.md) |
| Modify agent loop semantics | `packages/agent/src/agent-loop.ts` — see [`packages/agent/AGENTS.md`](packages/agent/AGENTS.md) |
| Modify renderer | `packages/tui/src/tui.ts` `doRender()` — see [`packages/tui/AGENTS.md`](packages/tui/AGENTS.md) |
| Document fork mod | new section in nearest existing `changes.md` (or create one alongside the modified file) |
| Run live-API tests | `./pi-test.sh` (gated by env vars; default `npm test` skips them) |

## FORK STRATEGY (CRITICAL)

Three-layer decision hierarchy before touching any upstream file:

1. **Can a builtin extension do it?** → add under `packages/coding-agent/src/core/extensions/builtin/<name>/` and register in `builtin/index.ts`.
2. **Can an external/user extension do it?** → ship under `packages/coding-agent/examples/extensions/` instead.
3. **Must modify upstream source?** → modify, then add a section to the nearest `changes.md` with: *what changed*, *why*, *why extension system couldn't handle it*, *expected merge-conflict zones*.

**`changes.md` is the merge-rebase contract.** Every fork-modified subdirectory has one. Tracked locations (do NOT remove):

- `packages/ai/{,src/,src/tool-call-middleware/}changes.md`
- `packages/agent/src/changes.md`
- `packages/coding-agent/{,src/,src/cli/,src/utils/}changes.md`
- `packages/coding-agent/src/core/{,compaction/,dynamic-prompt/,tools/}changes.md`
- `packages/coding-agent/src/core/extensions/{,builtin/{agent-system,compaction,permission-system,prompt-preset}/}changes.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [code-yeongyu/senpi](https://github.com/code-yeongyu/senpi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
