---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **dual-host plugin marketplace** shipping the same sdd (spec-driven development) workflow to two hosts: a **Claude Code plugin** (repo root, `/spec:x` commands) and an **OpenAI Codex CLI plugin** (`codex/`, `$spec-x` skills). **There is no application code, no compile/build/test runner** — the deliverables are markdown prompts, JSON/TOML manifests, hook scripts, and ast-grep rule packs. "Testing" means regenerating, loading the plugin in the real host, and watching a gate actually block.

**This repo dogfoods sdd on itself**: large changes go through the full research → propose → HARD GATE → apply → verify → archive flow. The resulting `spec/` artifacts are **gitignored** (local iteration records, never shipped or committed).

## Single source of truth: core/ (edit core/, never the generated trees)

All shipped markdown in BOTH plugins is generated from **`core/`** by `node tools/generate.mjs`:

| core/ source | Claude tree (near-identity) | Codex tree (6-rule transform) |
|---|---|---|
| core/commands/*.md | commands/*.md | codex/skills/spec-*/SKILL.md |
| core/skill.md | skills/core/SKILL.md | codex/skills/spec-core/SKILL.md |
| core/references/* | skills/core/references/* | codex/skills/spec-core/references/* |
| core/rules/* | rules/* | codex/skills/spec-core/rules/* |
| core/agents/*.md | agents/*.md | codex/agents/*.toml |

- The six mechanical Codex rules: sigil `/spec:x`→`$spec-x` · frontmatter mapping (drop `allowed-tools`/`model`/`color`/`tools`, add `name`) · reference-path rewrite · `--codex` section drop · host-marker selection · agent md→TOML (`developer_instructions = '''body'''`).
- Host-divergent passages use paired `<!-- host:claude -->` / `<!-- host:codex -->` … `<!-- /host -->` blocks (no nesting; unclosed marker or a `'''` in an agent body = generator **hard error**, never a silent skip — same for a missing core source file).
- Generated files carry a first-line/post-frontmatter `GENERATED from core/...` marker. The marker sits **after** YAML frontmatter in .md files (before it would break frontmatter parsing on both hosts) and as a `#` comment on line 1 of .toml/.yml.
- **NOT generator-owned** (hand-maintained): all hooks (`hooks/`, `codex/hooks/`), all manifests, READMEs, install scripts, `codex/skills/spec-setup`, `scripts/`.

## Dev loop (no build/test — regenerate, load, run for real)

```pwsh
# 1. Edit core/ (or a hand-maintained file), regenerate both trees
node tools/generate.mjs
# 2. Local Claude testing: the source copy wins over the marketplace cache
claude --plugin-dir .
# 3. Release: drift check → bump versions in THREE manifests → push
node tools/generate.mjs --check   # nonzero = hand-edit in a generated file or core changed without regenerating
#    .claude-plugin/plugin.json + .claude-plugin/marketplace.json + codex/.codex-plugin/plugin.json
#    (version drives cache refresh on BOTH hosts — Codex caches under <name>/<version>/)
git add . ; git commit -m "..." ; git push
# 4. Update local installs
claude plugin marketplace update spec-workflow
codex plugin marketplace upgrade spec-workflow    # Git-marketplace installs; local-path installs need remove+add instead
```

- **Claude hooks reload via `/reload-plugins`** (verified for the UserPromptSubmit gates + Stop reminder; monitors are the documented exception needing a restart). Commands / skills / agents hot-reload on their own. The old "hooks need a full restart" doctrine is obsolete for current Claude Code — but sessions started BEFORE a plugin update still run the old hook config until `/reload-plugins` or restart, which is the easiest trap in the repo.
- **Codex hooks require re-trusting in the TUI whenever hooks.json changes** (trust is recorded per hook-definition hash). Untrusted hooks are **silently skipped** — no error, no log. After any hook-affecting update, verify the gate bites: in a project with no `spec/changes/`, send `$spec-apply` — it must come back blocked.
- Codex agents aren't plugin-bundled (no such mechanism); `$spec-setup` copies the shipped TOMLs to `~/.codex/agents/` — re-run it after agent changes.

Validate all JSON manifests (no CI — run by hand):
```pwsh
Get-ChildItem -Recurse -Filter *.json | ForEach-Object { Get-Content $_.FullName -Raw | ConvertFrom-Json | Out-Null; "OK: $($_.Name)" }
```

## Manifests (four, roles differ)

- `.claude-plugin/marketplace.json` (`source: "./"` — repo root IS the Claude plugin root) + `.claude-plugin/plugin.json` — the Claude pair, keep name/version/description in sync.
- `.agents/plugins/marketplace.json` — the Codex marketplace (plugin `spec` → `./codex`).
- `codex/.codex-plugin/plugin.json` — the Codex plugin (bundles `skills` + `hooks` keys; Codex also falls back to reading `.claude-plugin/plugin.json`, but we ship both explicitly).

## Big picture: soft vs hard constraints

The whole design centers on "**stopping for real where you have to stop**":

1. **Soft constraints** (prompt text saying "you must do X") — the model can violate them.
2. **Hard constraints** (hook scripts that intercept) — a 0% violation rate.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kamioj/spec-workflow](https://github.com/kamioj/spec-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
