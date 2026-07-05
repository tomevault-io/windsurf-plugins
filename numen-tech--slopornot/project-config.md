---
trigger: always_on
description: Brief for AI coding agents (Claude Code, Codex, Cursor, Gemini CLI, Aider) editing this repo. The runtime skill itself is `skills/agentic-humanizer/SKILL.md`; this file is for agents working *on* the repo, not running the skill.
---

# SlopOrNot: agent guide

Brief for AI coding agents (Claude Code, Codex, Cursor, Gemini CLI, Aider) editing this repo. The runtime skill itself is `skills/agentic-humanizer/SKILL.md`; this file is for agents working *on* the repo, not running the skill.

## What this repo is

SlopOrNot is a plugin bundle for assistant workflows built around Slop or Not.
It ships two skills. `agentic-humanizer` runs a full 5-pass humanization
workflow with saved preferences and optional voice matching. It works without
Slop or Not; Slop or Not Pro only adds on-device AI detector scoring,
readability checks, Text Cleanup before and after humanization, and cleanup
stats. `slop-check` is a
self-contained one-shot router for the same on-device tools (text and image
detection, readability, cleanup, status) with no interview and no harness
routing files.

## Layout

| Path | Role |
|---|---|
| `skills/agentic-humanizer/SKILL.md` | Self-contained `agentic-humanizer` orchestrator. Steps 1-7 (harness detect, profile commands, preferences, voice, optional Slop probe, loop, output). |
| `skills/agentic-humanizer/harnesses/{claude-code,codex,cursor,gemini-cli,opencode,generic}.md` | Per-harness interview protocols. Edit only the file for the harness you're targeting. |
| `skills/agentic-humanizer/references/patterns.md` | 33-pattern rewrite vocabulary. |
| `skills/agentic-humanizer/references/detection-guidance.md` | English-only false-positive guard: what not to flag, and human-writing signals to preserve. Loaded on English runs alongside `patterns.md`. |
| `skills/agentic-humanizer/references/supplemental-ai-tells.md` | SlopOrNot-authored supplemental AI-tell checks inspired by Wikipedia's field guide. Language-agnostic S1 to S8 concepts, loaded on every run. |
| `skills/agentic-humanizer/references/multilingual.md` | Multilingual readability registry: supported languages, BCP-47 variants, readability formula per language, reading-level band mapping, code normalization (Norwegian Bokmal to `nb`). Single source of truth for non-English runs. |
| `skills/agentic-humanizer/references/profile-resolution.md` | Decision table for `SKILL.md` Step 3 rule 3: how a saved profile resolves against an unambiguously detected different language (language, variant, reading level, tone, length, English `target_grade`). Loaded only on the saved-profile-versus-detected-language path. |
| `skills/agentic-humanizer/references/ai-tells/<code>.md` | Per-language AI-tell catalogues (es, de, it, sv, da, no). Loaded when the resolved language is not English. `no.md` covers both Bokmal and Nynorsk. |
| `skills/agentic-humanizer/references/per-iteration-strategies.md` | The 5-iteration cookbook for Core mode and Slop or Not Pro, plus mid-flight Pro-gate fallback. |
| `skills/agentic-humanizer/references/voice-fingerprint.md` | Voice sample policy, fingerprint schema, extraction prompt, cache rules, and loop injection contracts. |
| `skills/agentic-humanizer/references/slop-{cli,mcp}-setup.md` | User-facing install guides. |
| `skills/agentic-humanizer/examples/sample-ai-text.md` | Smoke-test fixture (English). |
| `skills/agentic-humanizer/examples/sample-ai-text-de.md` | German smoke-test fixture (non-English path). |
| `skills/agentic-humanizer/README.md` | Dedicated Agentic Humanizer README for users and search indexing. |
| `skills/slop-check/SKILL.md` | Self-contained `slop-check` orchestrator. Steps 1-5 (identify op, resolve backend MCP/CLI/app-bundle fallback, run, format, fallback). |
| `skills/slop-check/references/slop-tools.md` | Full CLI + MCP tool surface for `slop-check`: params, flags, JSON field paths, score normalization, Pro-gating. |
| `skills/slop-check/references/slop-setup.md` | `slop-check` install, Pro unlock, app-bundle fallback, MCP/CLI registration. |
| `skills/slop-check/README.md` | Dedicated Slop Check README for users and search indexing. |
| `claude-skills/agentic-humanizer/SKILL.md` | Hand-authored Claude Desktop variant (no harness routing; built-in `ask_user_input_v0` interview). Canonical source for the Desktop build. |
| `claude-skills/agentic-humanizer/README.md` | Hand-authored Claude Desktop install/use guide. |
| `claude-skills/agentic-humanizer/{references,examples}/` | Copied verbatim from `skills/agentic-humanizer/` by `make -C claude-skills build`. Do not hand-edit. |
| `claude-skills/Makefile` | Builds and zips the Claude Desktop bundle (`build`, `zip`, `check`, `clean`). |
| `plugins/codex/slopornot/` | Generated Codex plugin payload. Do not edit synced skill files here by hand. |
| `plugins/claude/slopornot/` | Generated Claude Code plugin payload. Do not edit synced skill files here by hand. |
| `.agents/plugins/marketplace.json` | Codex repo marketplace for the `slopornot` plugin. |
| `.claude-plugin/marketplace.json` | Claude Code marketplace for the `slopornot` plugin. |
| `scripts/check-{frontmatter,links}.mjs` | Lint scripts run by CI. |
| `scripts/sync-plugins.mjs` | Copies canonical runtime files into plugin payloads, with `--check` drift detection. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [numen-tech/slopornot](https://github.com/numen-tech/slopornot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
