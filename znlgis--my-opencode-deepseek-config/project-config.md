---
trigger: always_on
description: These rules apply to every agent and load automatically as shared context;
---

# Global Operating Rules

These rules apply to every agent and load automatically as shared context;
`agents/*.md` only add what is unique to each role. When prompts overlap, follow
the stricter instruction. Routing tables, model tiers, and fallback chains live
in `agents/orchestrator.md` — do not restate them here.

## Core Principles

1. **Detect intent before acting.** "Look into X" is not "change X". Answer
   questions with analysis, not edits — never touch files unless the user
   explicitly asked for implementation.
2. **Make the smallest change that fully solves the task.** Don't touch
   unrelated code. A complete, correct solution beats a clever or broad one.
3. **Read before you write.** Never guess what code does — open it.
4. **Run independent work in parallel.** Fire multiple independent reads,
   searches, and fetches in a single batch.
5. **Respect role boundaries.** Read-only agents (`oracle`, `reviewer`,
   `explore`, `librarian`) never modify files; they report findings as text.
6. **Don't create files unless asked.** Never proactively create documentation,
   README files, or any new file without explicit user request.
7. **Right-size the model to the task.** Prefer flash for routing, search,
   lookup, planning, and routine implementation; reserve pro for deep
   reasoning, root-cause analysis, code review, and heavy multi-file
   implementation. When borderline, prefer flash, then escalate.
8. **Know your stop condition.** Before starting, define the observable
   condition that means "done". Once it holds and the change is verified,
   stop — no bonus polish or extra verification loops.
9. **Answer first, then act.** When the user asks a question, answer it before
   making edits or running implementation commands. When responding to user
   feedback, explicitly state whether you agree or disagree before saying what
   you changed.
10. **Be concise.** Keep answers short and direct. No fluff, no cheerful filler,
    no unnecessary preamble. Technical prose only.

## DeepSeek Cache & Thinking Discipline

- **Byte-stable prefix.** Agent prompts, AGENTS.md, and rule order stay
  byte-identical; early reorders bust the prefix cache and re-pay full input
  cost. Volatile content (timestamps, random IDs, per-request tokens, dynamic
  file lists) goes near the END of the payload, never the head.
- **Freeze toolsets.** Never reorder tool schemas or injected rules mid-session.
- **Temperature.** flash: 0 (thinking off). pro: unset — thinking is on and
  temperature/top_p are silently ignored.
- **Thinking.** flash = off (provider-level `thinking: {type:"disabled"}`, the
  official cost saver); pro = on (default). An agent may override per-agent via
  frontmatter `options.thinking` (e.g. `planner`/`light-orchestrator` re-enable
  thinking over flash's disabled default).
- **One-shot requests ride flash.** title/summary/compaction and other
  single-shot tasks run on flash so their volatile content never enters the
  pro prompt-cache prefix.
- **reasoning_content** must round-trip on tool calls (opencode handles this);
  never reorder messages in ways that break it.

### Thinking tiers

- **`reasoning_effort`** is a request-level thinking-strength control
  (`low`/`high`/`max`), NOT a model id — set per-agent via frontmatter
  `options` (camelCase `reasoningEffort`, deep-merged over `model.options`).
  The 2-model matrix is inviolate.
- **Tiers:** trivial (explore/librarian/consultant/ui-builder) = flash, thinking
  off; mid (planner/light-orchestrator) = flash, thinking on +
  `reasoningEffort: low`; deep (deep-worker/oracle/reviewer) = pro, default
  high. `solo` has no `model:` field, so it follows the session model and its
  thinking tier follows that model.
- **Routing:** trivial → flash off; routine-but-nontrivial multi-file → flash
  low; deep/uncertain → pro high.

## Scope First + Delegate Always

- **Size the scope first.** 2+ steps, multi-file, or architectural changes
  require `planner` first — never go straight to implementation.
- **BACKGROUND FIRST.** Independent subtasks dispatch in parallel, background.
  Requires `OPENCODE_EXPERIMENTAL_BACKGROUND_SUBAGENTS=true`; when unset,
  dispatch foreground instead.
- **Delegate, don't do.** Delegate whenever delegation overhead is smaller than
  the task; top-level tokens go only to routing and hard problems.
- **Subagent empty-result fallback.** A subagent returns an empty result with no
  workspace changes → retry once with a smaller task; if it fails again, stop
  and tell the user the subagent infrastructure is failing. Never retry the same
  task repeatedly, and never inline-execute a heavy implementation at the
  orchestrator level.
- **Pass the explicit `task_id`** when resuming a subagent session.
- **Reference paths, don't paste files.** Point at `src/app.ts:42`.

## Language

Reply to the user in the OS locale language (detect from environment). On a
zh-CN Windows system, Chinese; en-US, English. Never force English unless asked.

## Constraints (this repository)

- **No new models.** Only `deepseek/deepseek-v4-pro` and the natively
  multimodal `deepseek/deepseek-flash` may be used. Do not introduce others.
- **Vision input is opt-in.** `deepseek-flash` accepts images, but an image

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [znlgis/my-opencode-deepseek-config](https://github.com/znlgis/my-opencode-deepseek-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
