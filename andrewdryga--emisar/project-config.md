---
trigger: always_on
description: `emisar` is a control plane for AI-safe infrastructure actions: operators — and LLMs via MCP — dispatch **gated, audited** actions to a fleet of on-host runners. **It is a security product.** Treat every surface that ingests runner / LLM / operator input as hostile until proven otherwise.
---

# emisar — how we work (canonical agent manual)

`emisar` is a control plane for AI-safe infrastructure actions: operators — and LLMs via MCP — dispatch **gated, audited** actions to a fleet of on-host runners. **It is a security product.** Treat every surface that ingests runner / LLM / operator input as hostile until proven otherwise.

This file is the **canonical, tool-neutral operating manual** for any AI agent (Claude Code, Codex, …) working in this repo. `CLAUDE.md` is a symlink to it. Read it top to bottom — it is deliberately small; the deep, per-language rules live in each project's own `AGENTS.md`.

---

## ⟢ BOOT — read this when you start, or lose context

Context compaction drops everything except this file (re-injected from disk) and your tool's memory. When you start fresh, resume after a compaction, or feel unsure what you were doing, **re-read in this order before touching code**:

1. **This file** — the creed + the contract below.
2. **The project's `AGENTS.md`, in full** — not a skim. The rules are non-negotiable and you *will* violate them from memory.
3. **`.agent/kb/README.md`'s index** — the durable-knowledge routing table. Open a descriptive card, specification, runbook, or rule only when your task touches its subsystem or operation; never bulk-load the KB. Everything outside `kb/internal/` is customer-safe/public. For marketing, positioning, launches, company operations, or other non-customer-facing work, use the gitignored `kb/internal/` subtree and only the relevant material already there.
4. **`<project>/.agent/tasks/`** — the work queue (run `coop tasks`; it reads `.agent/tasks/`). Resume the first todo or in_progress task: read its `state.md` FIRST (the overwritten resume snapshot — where it stopped, the next action, traps), then its `log.md` (the append-only why-journal). That pair is how intent survives a compaction or a fresh box.

Five top-level areas, each with its own `AGENTS.md`:

| Project | Language | What it is | Read before editing |
|---|---|---|---|
| `portal/` | Elixir / Phoenix | the control plane (web, MCP, policy, approvals, audit, billing) | `portal/AGENTS.md` |
| `runner/` | Go | the on-host runner that executes actions | `runner/AGENTS.md` |
| `mcp/` | Go | the stdio↔HTTP MCP bridge for LLM clients | `mcp/AGENTS.md` |
| `packs/` | YAML | the action-pack catalog — what runners may execute | `packs/AGENTS.md` |
| `infra/` | Terraform | the Google Cloud production stack (compute, Cloud SQL, LB, DNS, secrets, monitoring) | `infra/AGENTS.md` |

---

## The creed — how we build (every project)

Generalize from these; defer to the project `AGENTS.md` for specifics.

1. **Pragmatic. Boring solutions that work.** Reach for the dull, proven shape first. Clever earns its place only when boring genuinely can't do the job — and you can say *why* in one sentence. No frameworks-within-the-framework, no speculative abstraction.
2. **We ship a good, _sellable_ product.** Every change serves a real operator need. Wear the hats before writing code — **PM** (is this the right thing to build, or should it be cut or smaller?), **UX** (is the operator's path obvious; are the empty / error / loading / offline states handled?), **Security** (what's the abuse case?), **Marketing** (does this make the product easier to explain and sell?), **Maintainer** (will this read clearly in six months?). Care about product, quality, UX, and marketing *genuinely* — but pragmatically: a half-built "platform" sells nothing; a small thing done well does.
3. **Readable code, no bloat.** Match the surrounding style exactly. Delete more than you add when you can. No config knobs nobody asked for, no "just in case" parameters, no comments narrating *what* — comments say *why*.
4. **Done means verified, not done-once.** A change is done when it compiles clean, is formatted, passes its gate, is covered by tests (including the denial / abuse path where it applies), and is obvious — not when it ran once. Never say "should work"; show the gate output, or state plainly what you could not verify.
5. **Verify APIs before you call them.** Never invoke a function / flag / option you are not sure exists with that signature. Check the repo, the dependency source, the docs — *first*. A hallucinated call costs far more than the 20-second check.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndrewDryga/emisar](https://github.com/AndrewDryga/emisar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
