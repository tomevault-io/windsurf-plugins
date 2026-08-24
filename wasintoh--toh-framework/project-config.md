---
trigger: always_on
description: Guide for a Claude Code agent developing the framework itself. Repo-only: not in the package.json
---

# CLAUDE.md — Toh Framework (repo development guide)

Guide for a Claude Code agent developing the framework itself. Repo-only: not in the package.json
`files` whitelist, so it never ships to npm. (claude-code.js generates a separate, unrelated CLAUDE.md inside end-user projects.)

## What this is

Toh Framework ("Type Once, Have it all") is the npm package `toh-framework` (v2.1.0, MIT, ESM,
Node >= 18, no build step) that installs an AI-orchestration development system into 6 IDEs:
Claude Code, Cursor (2.4+), Antigravity (agy CLI + IDE), Codex CLI, ZCode (Z.ai), and —
Enterprise-only, behind `--legacy-gemini` — Gemini CLI (consumer service shut down 2026-06-18).

North Star: a non-technical person types one sentence, approves once ("Go"), and THE TOH LOOP
builds, tests, and fixes a real, beautiful app until verified DONE. Quality is measured at the END
USER's experience; any added question or step is a regression — /toh-vibe asks zero questions, /toh-plan asks exactly one.

## Everyday commands

All npm scripts wrap `node bin/toh-cli.js <cmd>`, which lazy-loads `installer/*.js`:

- `npm run install:local` — interactive installer; `-- --quick` (plus `-t <dir>`) for non-interactive
  runs (dir must pre-exist; reinstalls still prompt). Flags: `--legacy-gemini`, `--legacy-cursorrules`.
- `npm run uninstall:local` — the counterpart; `-- -t <dir> --dry-run` to preview, `--yes` for
  scripts, `--all` to also delete .toh/plan.md + .toh/progress.md + both memory folders. Defaults to
  cwd, so ALWAYS pass `-t` when testing.
- `npm run list` — print the catalog, live-read from src/ frontmatter (throws on bad YAML)
- `npm run status` — inspect install state (~/.claude, ./.toh, manifest.json)
- `npm run bundle` — web prompt bundles into ./dist/web-bundles
- `npm pack --dry-run` — check exactly what ships before any packaging change

## Verification protocol

There is NO automated test suite — .github/workflows/ci.yml only smoke-tests the CLI. Verify by running for real:

1. Run what you touched — install into a scratch dir, `npm run list`/`status`, `npm pack --dry-run`.
   Inspect the generated output (.toh/, .claude/, .cursor/rules/ + .cursor/agents/, AGENTS.md +
   .codex/config.toml, .agents/skills + .agents/commands + legacy .agent/workflows/ mirror;
   .gemini/ only under --legacy-gemini) — never assume a transform worked.
   For ZCode, ask the runtime instead of reading files: `zcode skills list --cwd <dir> --json`
   must report 37 entries at scope "project" and `zcode commands list --cwd <dir> --json` all 14,
   both with an empty diagnostics array. The CLI ships inside the app
   (/Applications/ZCode.app/Contents/Resources/glm/zcode.cjs) and is usually not on PATH.
2. Coffee-Shop-Owner Test for any user-facing change: could a coffee-shop owner use this without
   tech vocabulary, never face an unanswerable question, know what to do when it breaks — and does the output look professionally made?

## Single source, transformed per IDE

`src/` is the ONLY source of truth; installed copies (.toh/, .claude/, .cursor/, AGENTS.md,
.agents/, .agent/, GEMINI.md) are generated output — editing them is always wrong. Edit `src/`, re-run the installer.

`installer/install.js` first copies shared resources into the target's `.toh/` (skills, agents,
commands, templates, 7 memory files, plan.md, progress.md, manifest.json, capabilities.json),
then 6 handlers in `installer/ide-handlers/` cover the IDEs. When cursor/codex/antigravity/zcode is
selected, shared.js writeAgentsSkills() also writes `.agents/skills/` — 37 wrappers (23 skills,
descriptions terse-capped at 300 chars for Codex's 8,000-char listing budget, + 14 toh-* command
skills converted from the TOML; throws on unparseable sources):

- claude-code.js → .claude/ + project CLAUDE.md + Stop hook + .claude/loop.md; agent frontmatter
  filtered to native keys (name/description/tools/model + autonomy pass-throughs; model defaults to
  sonnet; triggers dropped; `skills` passes through filtered by isPreloadableSkill — must exist in
  .claude/skills/, not disable-model-invocation); real alias command files generated from each
  command's aliases list. NEVER inject a default tools list — absence = unrestricted.
- cursor.js → .cursor/rules/*.mdc + native subagents .cursor/agents/*.md (keeps
  name/description/model; derives readonly:true when a present tools allowlist has no write
  tools — today only root-cause-debugger); root .cursorrules ONLY with --legacy-cursorrules.
- antigravity-cli.js → workspace .agents/: rules/toh-framework.md (Always-On, <=12,000 chars
  asserted), workflows/ (+ legacy .agent/workflows/ mirror), agents/*.md file-based subagents
  (subagent: true), deterministic Stop hook in .agents/hooks.json. Never writes .gemini/.
- codex.js → one root AGENTS.md between TOH-FRAMEWORK-START/END markers — compact agent roster +
  indexed command table; bodies read at runtime from .toh/. Hard-asserts the block <=24 KiB (Codex
  silently truncates at 32 KiB) and emits .codex/config.toml raising project_doc_max_bytes, never overwriting an existing one.
- zcode.js → thin by design. ZCode reads the same open surfaces, so it writes NO `.zcode/`: it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wasintoh/toh-framework](https://github.com/wasintoh/toh-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
