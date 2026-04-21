---
trigger: always_on
description: Generates static skill files for external tools (Cursor, VSCode Copilot, Gemini CLI, etc.). Run `bun run playground:generate`. Generators live in `.playground/<tool-name>/`, fixtures in `.playground/_fixtures/`, snapshots in `.playground/_snapshots/`.
---

# vercel-plugin Development Guide

## Quick Reference

- **Build hooks**: `bun run build:hooks` (compiles `hooks/src/*.mts` → `hooks/*.mjs` via tsup)
- **Build manifest**: `bun run build:manifest` (generates `generated/skill-manifest.json` from SKILL.md frontmatter)
- **Build from skills**: `bun run build:from-skills` (compiles `*.md.tmpl` → `*.md` by resolving `{{include:skill:…}}` markers)
- **Check from skills**: `bun run build:from-skills:check` (verify generated `.md` files are up-to-date; exits non-zero on drift)
- **Build all**: `bun run build` (hooks + manifest + from-skills)
- **Test**: `bun test` (typecheck + 32 test files)
- **Single test**: `bun test tests/<file>.test.ts`
- **Typecheck only**: `bun run typecheck` (tsc on hooks/tsconfig.json)
- **Validate skills**: `bun run validate` (structural validation of all skills + manifest)
- **Doctor**: `bun run doctor` (self-diagnosis: manifest parity, hook timeouts, dedup health)
- **Update snapshots**: `bun test:update-snapshots` (regenerate golden snapshot baselines)
- **Playground**: `bun run playground:generate` (generate static skill files for external tools)

Run `bun run build:hooks` after editing any `.mts` file. A pre-commit hook auto-compiles when `.mts` files are staged.

Run `bun run build:from-skills` after editing any skill referenced by a `.md.tmpl` template. The `build` script includes this step automatically.

## Architecture

### Hook Registration (`hooks/hooks.json`)

All hooks are registered in `hooks/hooks.json` and run via `node "${CLAUDE_PLUGIN_ROOT}/hooks/<file>.mjs"`. Hook output is type-checked against `SyncHookJSONOutput` from `@anthropic-ai/claude-agent-sdk`.

| Event | Hook | Matcher | Timeout |
|-------|------|---------|---------|
| SessionStart | `session-start-seen-skills.mjs` | `startup\|resume\|clear\|compact` | — |
| SessionStart | `session-start-profiler.mjs` | `startup\|resume\|clear\|compact` | — |
| SessionStart | `inject-claude-md.mjs` | `startup\|resume\|clear\|compact` | — |
| SessionEnd | `session-end-cleanup.mjs` | — | — |

### Hook Source Files (`hooks/src/*.mts`)

Source lives in `hooks/src/*.mts` (TypeScript) and compiles to `hooks/*.mjs` (ESM, committed).

**Entry-point hooks** (wired in hooks.json):
- `session-start-seen-skills.mts` — initializes `VERCEL_PLUGIN_SEEN_SKILLS=""` in `CLAUDE_ENV_FILE`
- `session-start-profiler.mts` — activates only for greenfield directories or detected Vercel/Next.js projects, then scans config files + package deps → sets `VERCEL_PLUGIN_LIKELY_SKILLS` (+5 priority boost)
- `inject-claude-md.mts` — outputs the thin session-start Vercel context plus knowledge update guidance for that same activation set
- `session-end-cleanup.mts` — deletes session-scoped temp files

**Library modules** (imported by entry-point hooks):
- `hook-env.mts` — shared runtime helpers (env parsing, path resolution)
- `skill-map-frontmatter.mts` — YAML parser + frontmatter extraction + `buildSkillMap()` + `validateSkillMap()`
- `patterns.mts` — glob→regex conversion, seen-skills helpers, ranking, atomic file claims
- `prompt-patterns.mts` — prompt signal compiler + scorer (phrases/allOf/anyOf/noneOf)
- `prompt-analysis.mts` — dry-run analysis reports for prompt matching
- `vercel-config.mts` — vercel.json key→skill routing (±10 priority)
- `logger.mts` — structured JSON logging to stderr (off/summary/debug/trace)

### Skill Injection Flow

1. **SessionStart**: For greenfield directories or detected Vercel/Next.js projects, the profiler scans the project → sets `VERCEL_PLUGIN_LIKELY_SKILLS`
2. **PreToolUse** (on Read/Edit/Write/Bash): Match file paths (glob), bash commands (regex), imports (regex+flags) → apply vercel.json routing → apply profiler boost → rank by priority → dedup → inject up to 3 skills within 18KB budget
3. **UserPromptSubmit**: Score prompt text against `promptSignals` (phrases/allOf/anyOf/noneOf) → inject up to 2 skills within 8KB budget
   - **3b. Lexical fallback** (when `VERCEL_PLUGIN_LEXICAL_PROMPT=on`): If phrase/allOf/anyOf scoring yields no matches above `minScore`, re-score using a lexical stemmer that normalizes prompt tokens before comparison — catches natural phrasing that exact-substring matching misses
4. **SessionEnd**: Clean up session-scoped temp files

Special triggers in PreToolUse:
- **TSX review**: After N `.tsx` edits (default 3), injects `react-best-practices`
- **Dev server detection**: Boosts `agent-browser-verify` when dev server patterns appear
- **Vercel env help**: One-time injection for `vercel env` commands

### Skill Structure (`skills/<name>/SKILL.md`)

43 skills in `skills/`. Each has a `SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-slug
description: "One-line description"
summary: "Brief fallback (injected when budget exceeded)"
metadata:
  priority: 6                    # 4-8 range; higher = injected first
  pathPatterns: ["glob1"]        # File glob patterns
  bashPatterns: ["regex1"]       # Bash command regex patterns
  importPatterns: ["package"]    # Import/require patterns
  promptSignals:                 # UserPromptSubmit scoring

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel/vercel-plugin](https://github.com/vercel/vercel-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
