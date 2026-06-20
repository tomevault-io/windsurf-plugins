---
trigger: always_on
description: Use this skill to offload non-reasoning text work to locally-installed models (Ollama or MLX) via `delegate.sh`, keeping content on-device and freeing the main-agent context window. Saves API tokens too, though context protection and privacy are the headline values, not cost. MUST use whenever the user asks to summarise a log/diff/file/PR/issue, draft a commit message/changelog/release note, triage or classify many items, extract structured fields from free text, skim many files for a one-liner,
---


# Delegate Local

Offload non-agentic text tasks to locally-installed models via the local inference backend. The headline value is privacy (content stays on-device) and context protection (the main-agent window is not consumed by paragraph-fills); token cost savings are real but typically pennies per session and a side effect, not the reason to reach. Local models are strong summarisers and weak reasoners — scope accordingly.

**Core insight (from local-brain):** you do not need a framework. You need `context | ollama run model`.

## Operating mode

Default to auto-delegate. When this skill is loaded into the conversation and the user's task matches the Fits list below, delegate immediately without asking permission. After every successful `delegate.sh` call, read the `delegate-meta:` line emitted to the tool's stderr — it carries `model`, `tier`, `backend`, `tokens_local`, and `duration_ms` as space-separated `key=value` pairs (plus `recipe="NAME"` when a `--recipe` flag was used). String-typed fields are double-quoted (`model="qwen3.6:35b-a3b-q8_0"`); integer fields are bare. In your reply, surface the model and the local-token count so the user can both spot a bad answer and see what stayed on-device: e.g. "Delegated to qwen3.6:35b-a3b-q8_0 (prose tier) — ~578 tokens kept local in 1.4 s." Frame it as "kept local," not "saved from Claude" — `tokens_local` is the local model's tokenizer view of total chars in + out divided by 4 (the same value `scripts/metrics-summary.sh` rolls up under `estimated_tokens_avoided`), not Anthropic's billable count. Do not ask "should I delegate this?" — the skill being loaded means yes by default. Re-confirm only if the task is borderline or the input contains content the user explicitly flagged as sensitive.

If the user says any of: "delegate where it fits", "use ollama where appropriate", "auto-delegate", "route to local where it makes sense", or similar — treat that as durable consent for the rest of the conversation: from that point on, every matching task goes through `ollama run` without further prompting.

## When to delegate

Fits:
- Summarise a long log, diff, file, or PR description.
- Draft a commit message, changelog entry, or release note for a single-file mechanical change.
- Classify or triage N items (relevant/noise, bug/feature).
- Closed-form classification with an explicit, finite option set (e.g. "high|medium|low|info" severity, "REAL|FP" filter against a stated allowlist). The 2026-05-03 retrospective measured 5/5 reps perfect on FP-filter when the allowlist rule was explicit and a one-shot example was included.
- Compose structured prose from a fixed list of items (PR comment from a findings list, release-note bullets from a changelog block). The 2026-05-03 retrospective measured 3/3 PASS on PR-comment composition once the prompt forbade placeholder substitution explicitly.
- First-pass "what does this file do" over many files.
- Extract structured fields (JSON) from free-form text.
- Reformat or rewrite prose.
- Minimal single-file code patches where you supply the failing test alongside the source and verify by re-running the test after applying the patch. Scope is narrow and the verification step is load-bearing — multi-file edits, cross-module reasoning, and tests the director cannot re-run are out of scope. See the v8 probe evidence and the director-side pattern in the Reasoning-tier and Discipline sections.

Do NOT delegate:
- Multi-step reasoning, planning, or tool-calling.
- Tasks needing repo-wide context that does not fit one prompt.
- Code correctness, security, architectural judgements.
- Anything whose output directly triggers a destructive or shared-state action without review.
- The user asked for *your* analysis specifically.

The natural shape is **structure is yours, prose is delegated**: headings, code blocks, file links, table layouts, and CLI examples are decisions to own; flowing paragraphs of explanatory text are what the model fills. Tasks that look textual but encode structural decisions (link-index rewrites, table moves, cross-reference collapses, "delete this section and replace with a pointer to the canonical home") are not delegation candidates even though their output is text — the work is deciding which home is canonical, not writing prose.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IsmaelMartinez/delegate-local](https://github.com/IsmaelMartinez/delegate-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
