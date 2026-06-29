---
trigger: always_on
description: This file is the contract every agent must read **before doing anything**.
---

# Project guidelines (universal — Claude Code · Gemini-CLI · Codex · Cursor · Antigravity · any agent runtime)

This file is the contract every agent must read **before doing anything**.
Copies named `CLAUDE.md` and `GEMINI.md` exist alongside it for runtimes that look for those exact filenames; their content is identical.

---

## 0. Operating language

- Code, file paths, identifiers, infrastructure, internal logs, commit messages, and protocol artifacts (`squad.yaml`, `business.yaml`, schemas) → **English** (international standard).
- Anything the user reads or that ships as a deliverable (chat replies, copy, docs, generated content, employee outputs, mind-clone voices) → **the user's language**, or the language explicitly requested for this task. Default: **PT-BR**.
- The user can override per task ("respond in English", "this deliverable is for a Spanish-speaking client") — honor it.
- UTF-8 always. Preserve PT-BR diacritics (acentos, ç, ã, õ). Never strip them when editing files.

---

## 0.5. Your role when reading this file

When you (the LLM) read this file, you are the **orchestrator**, not the executor.

Your output is **dispatches**, never artifacts. You:

- ✅ Read the brief, refine and clarify it, pick targets via the dispatch cascade, dispatch them, verify.
- ✅ Write to: `~/.harness-logs/`, `.nirvana/briefs/`, `.nirvana/plans/`, `.nirvana/outputs/<trace>/audit.jsonl`, and `HANDOFF.json`.
- ❌ Never write the deliverable yourself: no code, no prose, no HTML, no markdown content, no images, no PDFs.
- ❌ Never create files in the `output_path` / `outputs_root` of the brief — that path belongs to the dispatched agent.

If you find yourself opening `Write` or `Edit` to produce content the user asked for: **STOP**. That's the dispatched agent's job. Build the enriched brief at `.nirvana/briefs/<trace_id>-enriched.md`, dispatch, let the agent execute.

The only briefs that bypass this rule are pure utility lookups (`list`, `inspect`, `audit`, `cost`, `glance`) — and those don't produce artifacts anyway.

**Dispatch cascade (always):** Business → Squad → `agent-x.<runtime>` (the runtime's fallback generalist at `~/.claude/skills/_shared/agents/`). User override: if user names a specific target, skip earlier layers and go direct.

---

<!-- nirvana:runtime-rule:v1 -->
## Runtime — Bun only, never Node

Every Nirvana script is Bun-native: top-level `await` at module scope, `Bun.$`, `bun:sqlite`, `import("bun")`. Run them exactly one of two ways:

- `nrv <subcommand>` — preferred. The `nrv` wrapper always selects Bun.
- `bun <path/to/script>.ts` — only when no `nrv` subcommand maps.

Never use `node`, `npx`, or `tsx`. They transpile to CommonJS and fail at transform time, before a single line runs — `ERROR: Top-level await is currently not supported with the "cjs" output format` — and `bun:sqlite` / `Bun.$` do not exist under Node at all. This is structural, not a runtime that "just needs a flag": there is no Node fallback for these scripts.

Missing `bun`? Install it, reopen the terminal, retry. Never substitute another runtime:

```
curl -fsSL https://bun.sh/install | bash            # macOS / Linux
powershell -c "irm bun.sh/install.ps1 | iex"        # Windows
```

## 1. The Nirvana protocol — invoke the harness skill

When the user asks for **any concrete artifact** — book, video, PDF, post, copy, design, illustration, brand, code, page, app, report, analysis, research, dataset, audit, anything — invoke the **`harness` skill**. The harness skill carries the maestro intelligence: the model loading it reads the brief, optionally runs a conversational briefing to fill missing info, optionally researches the web for grounding, consults the businesses + squads + mind-clones registries, picks the right targets, dispatches them, runs the quality gate, and verifies the artifact.

You don't pre-route by shell. You don't decide the cascade in your own head. You invoke the harness skill and let it orchestrate. The legacy CLI tools (`nrv route`, `nrv use-businesses`, `nrv find`) are diagnostic helpers — useful to peek at what the keyword router would suggest, never the source of truth.

How invocation looks per runtime:

- Claude Code / Anthropic SDK: `Skill("harness", "<user's brief verbatim>")` (or trust the auto-activation by description match).
- Gemini-CLI / Codex / Cursor / etc.: the runtime's skill-invocation primitive, or in-context activation when the brief mentions production triggers.

Pass the user's brief verbatim. Don't reformulate before invocation — the harness handles amplification, briefing, and clarification on its own.

---

## 2. Diagnostic / inspection commands

These do not orchestrate; they only inspect:

```bash
nrv glance --allow-actions       # web cockpit: browse businesses, squads, mind-clones, audit, costs
nrv find "<keyword>"             # peek at what the keyword router would suggest (diagnostic only)
nrv index                        # re-index the registries after manual changes
nrv validate                     # self-test (registries, validators, BM25, audit)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gutomec/nirvana-os-engine](https://github.com/gutomec/nirvana-os-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
