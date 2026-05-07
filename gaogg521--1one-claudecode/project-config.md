---
trigger: always_on
description: - **Directory size limit**: A single directory must not exceed **10** direct children (files + subdirectories). Split by responsibility when approaching this limit.
---

# 1ONE ClaudeCode - Project Guide

## Code Conventions

### File & Directory Structure

- **Directory size limit**: A single directory must not exceed **10** direct children (files + subdirectories). Split by responsibility when approaching this limit.

See [docs/conventions/file-structure.md](docs/conventions/file-structure.md) for complete rules on directory naming, page module layout, and shared vs private code placement. Agents working in this repository must also read and follow the `architecture` skill (`.claude/skills/architecture/SKILL.md`) when creating files, modules, or making structure decisions.

### Naming

- **Components**: PascalCase (`Button.tsx`, `Modal.tsx`)
- **Utilities**: camelCase (`formatDate.ts`)
- **Hooks**: camelCase with `use` prefix (`useTheme.ts`)
- **Constants files**: camelCase (`constants.ts`) — values inside use UPPER_SNAKE_CASE
- **Type files**: camelCase (`types.ts`)
- **Style files**: kebab-case or `ComponentName.module.css`
- **Unused params**: prefix with `_`

### UI Library & Icons

- **Components**: `@arco-design/web-react` — no raw interactive HTML (`<button>`, `<input>`, `<select>`, etc.)
- **Icons**: `@icon-park/react`

### CSS

- Prefer **UnoCSS utility classes**; complex styles use **CSS Modules** (`ComponentName.module.css`)
- Colors must use **semantic tokens** from `uno.config.ts` or CSS variables — no hardcoded values
- Arco overrides go in the component's CSS Module via `:global()` — no global override files
- Global styles only in `src/renderer/styles/`

See [docs/conventions/file-structure.md](docs/conventions/file-structure.md) for full CSS and UI library rules.

### TypeScript

- Strict mode enabled — no `any`, no implicit returns
- Use path aliases: `@/*`, `@process/*`, `@renderer/*`, `@worker/*`
- Prefer `type` over `interface` (per Oxlint config)
- English for code comments; JSDoc for public functions

### Architecture

Three process types — never mix their APIs:

- `src/process/` — main process, no DOM APIs
- `src/renderer/` — renderer, no Node.js APIs
- `src/process/worker/` — fork workers, no Electron APIs

Cross-process communication must go through the IPC bridge (`src/preload.ts`).
See [docs/tech/architecture.md](docs/tech/architecture.md) for details.

## Memory Conventions (Critical)

- **Memory = 摘要 + 路由**：记忆文件只记录“结论/约束/决策/常用入口”，并给出原始文档路径；不要把整份设计文档粘进记忆。
- **Source of truth**：需要细节时，按路由去读仓库内 `docs/**`（或业务仓库内的真实文档）；记忆只作为索引与缓存。
- **Where memory lives**：Claude Code 的自动记忆在 `~/.claude/projects/{project}/memory/*.md`；项目指令在 `.claude/CLAUDE.md`（优先）或仓库根 `CLAUDE.md`。
- **Project routing file**：每个项目都应有一个 `.claude/CLAUDE.md`，专门维护“关键文档入口表（路径 + 何时阅读）”。

## Testing

**Framework**: Vitest 4 (`vitest.config.ts`). Run `bun run test` before every commit. Coverage target ≥ 80%.

See the `testing` skill (`.claude/skills/testing/SKILL.md`) for complete workflow, quality rules, and checklist.

## Code Quality

**During development** — auto-fix as you edit:

```bash
bun run lint:fix       # auto-fix lint issues in .ts / .tsx (oxlint)
bun run format         # auto-format .ts / .tsx / .css / .json / .md (oxfmt)
bunx tsc --noEmit      # verify no type errors
```

**Before every PR** — run the full CI check locally to catch everything CI catches (end-of-file, trailing whitespace, all file types):

```bash
# One-time setup
npm install -g @j178/prek

# Replicate exact CI check (read-only — does not auto-fix)
prek run --from-ref origin/main --to-ref HEAD
```

> Note: `prek` uses `lint` (check only) and `format:check` (check only) — it will fail if there are issues but won't fix them.
> If prek reports formatting or lint issues, run the auto-fix commands above first, then re-run prek to verify.

**i18n validation:** If your changes touch `src/renderer/`, `locales/`, or `src/common/config/i18n`, run:

```bash
bun run i18n:types
node scripts/check-i18n.js
```

Both commands must complete without errors before opening a PR. The `oss-pr` skill enforces this automatically.

Common Oxfmt rules (Prettier-compatible, avoid a fix pass):

- Single-element arrays that fit on one line → inline: `[{ id: 'a', value: 'b' }]`
- Trailing commas required in multi-line arrays/objects
- Single quotes for strings

## Git Conventions

Commit format: `<type>(<scope>): <subject>` in English. Types: feat, fix, refactor, chore, docs, test, style, perf. **NEVER add AI signatures** (Co-Authored-By, Generated with, etc.).

For pull request creation, see the `oss-pr` skill (`.claude/skills/oss-pr/SKILL.md`).

## Skills Index

Detailed rules and guidelines are organized into Skills for better modularity:

| Skill             | Purpose                                                                              | Triggers                                                                  |
| ----------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------- |
| **architecture**  | File & directory structure conventions for all process types                         | Creating files, adding modules, architectural decisions                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gaogg521/1ONE-ClaudeCode](https://github.com/gaogg521/1ONE-ClaudeCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
