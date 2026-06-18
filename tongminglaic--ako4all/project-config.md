---
trigger: always_on
description: Drive an agentic loop that iteratively optimizes a GPU kernel for maximum speedup. Use this skill whenever the user wants to optimize / speed up / benchmark a GPU kernel (CUDA, Triton, TileLang, C++, Python), mentions AKO / AKO4ALL / AKO4X / agentic kernel optimization, asks to "make this kernel faster", or has a kernel they want measured against a PyTorch reference. The skill handles setup, profiling (ncu), correctness checking, iteration logging, and git commits. Bootstraps a workspace in any 
---


# AKO4ALL — Agentic Kernel Optimization

Drive a profile → modify → benchmark → log → commit loop on a GPU kernel until it runs faster than the reference. The user provides at minimum a kernel; everything else (reference, inputs, bench script, hints) is optional.

## When this skill applies

- "optimize this kernel" / "speed up this CUDA / Triton / TileLang kernel"
- "run AKO / AKO4ALL on ..."
- "benchmark this kernel against PyTorch"
- "iterate on this kernel until it's faster"
- mentions of `ncu`, kernel profiling, GPU speedup target

Does NOT apply when:
- User wants to *write* a new kernel from scratch with no optimization target — just write code, no loop.
- User wants Codex / GPT to review or implement — use `codex:rescue` instead.
- User wants generic performance advice for code that isn't a GPU kernel.

## First action

Before doing anything else, establish the **workspace** — the directory the loop runs in. It is typically the user's CWD, or a subdirectory / path they name in the prompt.

### Inventory the workspace + prompt

Browse the workspace (don't run a fixed checklist — look around) and read the user's prompt to identify what the loop needs:

- **Kernel** (required) — the code to optimize
- **Reference** (optional) — correctness golden
- **Input data** (optional) — data files the kernel consumes (`.npz`, `.bin`, shape lists, custom formats, etc.)
- **Knowledge** (optional) — reference materials the user wants you to consult: algorithm notes, papers, design docs, prior PRs. Typically under `knowledge/` but anywhere the user points at.
- **Bench mode** — user-provided bench script vs. default `bench/kernelbench/` evaluator
- **Scaffold presence** — whether `bench-wrapper.sh`, `HINTS.md`, `ITERATIONS.md`, `bench/kernelbench/` are already at workspace root

Whether the workspace follows AKO4ALL's `source/` / `knowledge/` / `bench/` naming or some entirely different layout is **not** the signal. What matters is whether you can identify each item above with confidence.

### Ask only when genuinely uncertain

If the user's prompt + filesystem give you confidence about every required item, **don't ask** — skip straight to presenting the plan. Ask only when a piece's role is genuinely ambiguous (a kernel-shaped file with no obvious reference, two files that could both be the kernel, an input data file in an unfamiliar format you need permission to wire up a custom way, etc.). When in doubt, asking is cheaper than guessing wrong.

### Always present the resolved plan before running anything

Whether you asked the user or not, list back what you decided — so the user can correct you even when you didn't think you needed to ask.

Use the format below. Bold field labels + inline-code path values + the leading emoji marker make the plan visually scannable in any terminal theme (don't flatten to a wall of prose):

**📋 Resolved Plan**

- **Workspace** — `<path>`
- **Kernel** — `<path>`
- **Reference** — `<path>` *(or none — will use original kernel)*
- **Input data** — `<path>` *(or inline in ref, or none)*
- **Knowledge** — `<path>` *(or none)*
- **Bench mode** — default (KernelBench) *(or custom: `<path>`)*
- **Scaffold to copy** — `<list of missing files>` *(or none — already present)*

If anything still feels uncertain at this point, **stop and ask**. Otherwise proceed to Workflow.

### Bringing in scaffold

When copying scaffold (`bench-wrapper.sh`, `bench/kernelbench/`, starter `HINTS.md` / `ITERATIONS.md`, `workspace.gitignore` → as `.gitignore` in the workspace) from this skill's own directory into the workspace, **do not overwrite** files that already exist — the user may have edited `HINTS.md`, or `ITERATIONS.md` may carry prior iteration history. Copy only what's missing.

### Persisting user-supplied hints

The user may supply behavior directives in two ways:
- **Inline in the prompt** — e.g., "do not use shared memory" or "prefer Triton".
- **External file reference** — e.g., "follow rules in /tmp/x.md" or "see hints.md".

In **both cases**, merge those directives into `HINTS.md`. It's the persistence
layer — directives that only live in the current session's plan are lost on resume.

### Surfacing HINTS.md changes

Whenever you merge directives into `HINTS.md`, tell the user explicitly what
happened. Example phrasings:

> "I added your 'avoid shared memory' directive from the prompt to HINTS.md."
> "I added the 3 rules from /tmp/user-hints.md to HINTS.md."

Without this acknowledgment the user can't tell from your reply whether you
added, replaced, or silently dropped their directives. Always name the **source**
("from your prompt" / "from /tmp/x.md").

## Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TongmingLAIC/AKO4ALL](https://github.com/TongmingLAIC/AKO4ALL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
