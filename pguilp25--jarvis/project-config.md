---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What JARVIS is

A multi-brain AI agent that coordinates many **free/weak** LLMs (NVIDIA NIM, OpenRouter, DeepInfra, Groq, Gemini) to approximate frontier-model quality at $0, slower. The bet: several models plan independently, critique, and merge, beating one weak model thinking alone. The hardest and most actively developed part is the **coding agent** — a 4-role pipeline (UNDERSTAND → PLAN → IMPLEMENT → REVIEW) whose target is to score well on SWE-bench / SWE-bench Pro with weak models.

## Parallel-instance coordination (READ FIRST — two Claude Code instances are active)

Two Claude Code instances work this repo **at the same time**. Stay in your lane; flag anything that crosses lanes **before** you touch it. Assume the other instance has unrelated edits in flight — never `git checkout`/revert/`stash` broadly, never "clean up" files you don't own, and `git pull`/rebase mindset: your working tree is shared.

- **Lane A — Coding agent** (the UNDERSTAND→PLAN→IMPLEMENT→REVIEW pipeline + its SWE-bench eval). Files: `core/native_tools.py`, `core/tool_call.py`, `core/tool_detector.py`, `core/prompts_v8.py`, `core/self_verify.py`, `core/structural_gate.py`, `core/remote_check.py`, `core/review_verify.py`, `core/verify_toolkit.py`, `core/plan_*.py`, `core/code_index.py`, `core/cli.py` (trace), `workflows/code.py`, `tools/`, `swe_bench.py` + `behavioral_audit/` (coding-agent eval/grading).
- **Lane B — Context system** (a NEW subsystem giving the MAIN PIPELINE better context; explicitly **NOT** the coding agent — used by the main pipeline, not the coder/planner/reviewer). Files: the new context-system module(s) + the main-pipeline hooks that gather/inject context. *(Lane B owner: list your files here.)*

**Shared / cross-cutting — touching these can impact the other lane → ANNOUNCE first (log line + commit note):**
`main.py`, `ui_main.py` and the top-level pipeline orchestration; `clients/` (the LLM clients both lanes call); shared config / env-key handling; `CLAUDE.md`; and — the one real seam — **the handoff where the main pipeline passes the task/context into the coding agent**. If Lane B changes *what context the coder/planner actually receives*, that DOES reach Lane A → flag it loudly (Lane A's whole doctrine is "verify the rendered artifact the model receives"; silently changing that artifact breaks it).

**Rules:**
1. Edit only your lane. Do NOT refactor, rename, move, or "fix" the other lane's files — if it looks wrong, leave it and flag it.
2. Before touching a SHARED file (or anything in the other lane), STOP and append a line to the **Cross-lane log** below (file · what · why). Prefer an ADDITIVE change (new function / new param with a default) over modifying a signature the other lane consumes.
3. If you change a shared signature / contract / data-shape, say so explicitly in the log AND the commit message.
4. Commit-message prefix: `[agent]` (Lane A) / `[ctx]` (Lane B), so the other can filter `git log`. Commit your own files; don't sweep the other's changes into your commit.
5. Run discipline: exactly ONE `swe_bench.py` run at a time (Lane A owns it); the grading PC (ngrok) is shared infra — don't launch work that competes with a run in flight.

**Cross-lane log** (append newest-first when you touch shared/cross-cutting):
- `CLAUDE.md` · [agent] added the REFLEX-vs-CoT doctrine bullet to the Engineering-doctrine Non-negotiables (Lane A prompt-engineering principle from the GW20l forensic). PURELY ADDITIVE (a new doctrine bullet — no code, no signature/contract change, does not touch Lane B). · WHY: a model-facing rule only BINDS as a conditional reflex when its trigger is surface-lexical + its check mechanical (S2 bound); inference-triggered / shallow-check / prerequisite-needing rules must be woven CoT (S1/S3 didn't bind) — codifying so future prompt work (both lanes' model-facing prompts, and every prompt-design subagent) picks the shape deliberately + doesn't bloat.
- `ui/index.html` + `ui/server.py` · [ctx] medium_chat ROUND-BY-ROUND UI revamp (Lane B). `run_turn` now takes an optional async `emit` callback that streams per-round events (`round_start` → `proposal` {role,name,thinking,tools} → `merge` {thinking,message,tools} → `tool` {tool,args,result} → `final`); loop now TERMINATES when a round runs no ACTION tool (its message is the answer); a proposer fallback RENAMES in place (`gpt-oss ·paid`), no new entity. `ui/server.py` broadcasts these as `mc_*` (mc_final renders the bubble; memory still persists). `ui/index.html` renders per-round collapsible blocks (each model + merger click-to-reveal thinking, the merger response, tools click-to-reveal full args/result), color-coded per role, and a guard at the top of `h()` DROPS the raw `thinking_*`/`status` stream while `mcLive[cvid]` is set (our blocks replace it). ADDITIVE — normal pipeline rendering untouched. · WHY: the user's requested transparent round-by-round chat UX.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pguilp25/jarvis](https://github.com/pguilp25/jarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
