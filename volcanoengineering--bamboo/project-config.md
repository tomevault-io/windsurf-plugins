---
trigger: always_on
description: You are Claude. You have landed in **Bamboo** — the canonical library of `.md` files (with the policy spec at `Bamboo.md`) that gets forked into projects to give AI agents a consistent set of standards across vendors. This file is your Claude-specific cold-start overlay. It sits on top of `AGENT.md`, not in place of it.
---

# CLAUDE.md

You are Claude. You have landed in **Bamboo** — the canonical library of `.md` files (with the policy spec at `Bamboo.md`) that gets forked into projects to give AI agents a consistent set of standards across vendors. This file is your Claude-specific cold-start overlay. It sits on top of `AGENT.md`, not in place of it.

Read `AGENT.md` first. It is vendor-neutral and tells you the operational order of operations on first contact. This file adds the Claude-shaped notes: how to use Skills the way Claude expects them, how to weigh repository memory against your context window, and where the Knob log lives in this repo.

If `AGENT.md` and this file disagree, `AGENT.md` wins. This file is an overlay, not an override.

---

## What this repo is

**Bamboo** is not a product (yet — the SaaS layer is coming). The framework itself is a documentation library — a fork-and-go starter kit of `.md` files that other repositories ingest to give their agents shared rules, shared vocabulary, and shared Skills regardless of vendor. The canonical policy spec lives in `Bamboo.md` at the repo root.

The repo is laid out across seven working folders plus `docs/`:

- `behavior/` — the rules an agent obeys. Context, memory, handoffs, Token economy. Cold-start required.
- `architecture/` — **advanced add-on.** Memory architecture layer (ADM, RAG, Memory, Drift, Watchdog, workflow tools). Skip unless your project explicitly has an ADM/RAG memory layer or you're auditing memory governance. Most projects don't need this folder.
- `development/` — implementation standards and engine specs (Unity, UE5, web, Next.js, React, Swift, generic app). Load only when building in that stack.
- `BAMBOO-OS extension (private)` — **advanced add-on.** Multi-agent identity, topology, and orchestration patterns. Skip unless your project has a multi-agent topology with handoff/orchestration boundaries. Single-agent projects don't need this folder.
- `skills/` — portable AI capabilities that work the same across Claude, Codex, Gemini, GPT, Copilot.
- `workflows/` — DevOps and project lifecycle patterns. Forkable, overridable.
- `design/` — project-specific UI/UX rules. Skip on cold start.
- `docs/` — this repo's own operational memory: the folder map and the per-Knob change log, with state logs under `docs/memory-ctx/`.

See `docs/repo-organization.md` for the full layout and what each file covers. Read that before assuming where something lives.

---

## Cold-start order for Claude

1. `README.md` — what this repo is and who it's for.
2. `AGENT.md` — the vendor-neutral cold-start file.
3. This file (`CLAUDE.md`) — the Claude overlay.
4. `behavior/ctx-rules.md` — foundational rules and the Knob entry format (5 shape variants).
5. `behavior/ctx-lexicon.md` — the decoder ring. Concepts (Knob, Bump, Entropy, Wayfinding, Decay, Drift, Bloat, Collapse, CTX) and operational acronyms (PLTRF, LTIP, STIP, CWM, CTL, ADM, RAG, CRUD). Load this when you hit a term you don't recognize. (CTX = Context, the shorthand prefix used across the `ctx-*.md` file family.)
6. `behavior/ctx-entropy.md` — the preservation view. PLTRF, LTIP, STIP, hot/warm/cold tiering. Read the worked examples; the vocabulary attaches to concrete moves there.
7. `behavior/ctx-window.md` (CWM) — the active memory view. Saturation, drift, compression.
8. `behavior/ctx-token-limits.md` (CTL) — the Token economy view. Scoring, wayfinding, conservation practices.
9. `behavior/ctx-utility.md` — the index for `behavior/`. Use it as a map, not a substitute for the docs.
10. **Advanced add-on — skip unless the task explicitly demands it:** `architecture/` — only if the project has an ADM/RAG memory layer or you're auditing memory governance / drift / Watchdog. Most projects don't need this folder.
11. `docs/memory-ctx/ctx-orientation.md` — what changed recently and why. This is the current Knob in narrative form.
12. `skills/skill-map.md` and any relevant `SKILL.md` under `skills/`.
13. `workflows/` — only if the task touches project setup or context governance.
14. **Advanced add-on — skip unless the task explicitly demands it:** `BAMBOO-OS extension (private)` — only if the project has a multi-agent topology with handoff/orchestration boundaries. Single-agent projects don't need this folder.
15. `development/` — only if the task is building in a specific stack (Unity, UE5, web, etc.). Load the matching spec, skip the rest.
16. `design/` — only if the task is design or UI work.

You do not need to load all of these into active context at once. Use the wayfinding discipline in `ctx-token-limits.md`: pull what the current task references, leave the rest cold. We are using hot, and cold to write to context memory.

---

## Claude-shaped operating notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VolcanoEngineering/Bamboo](https://github.com/VolcanoEngineering/Bamboo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
