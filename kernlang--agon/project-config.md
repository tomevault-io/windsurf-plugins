---
trigger: always_on
description: > Any AI can join. They compete. You ship.
---

# AGENTS.md — Agon AI

> Any AI can join. They compete. You ship.

Agon is a multi-AI orchestration framework. Registered engines compete via forge, debate via tribunal, and ideate via brainstorm. Cesar orchestrates from live capability, availability, confidence, and task evidence. Canonical cross-agent doctrine: `~/.claude/AGENTS.md`; Agon-specific global conventions: `~/.agon/AGON.md`.

## Using Agon From Codex

Codex agents should use Agon when the task benefits from multiple engines, adversarial review, team competition, or confidence-weighted ideation, but interactive chat must never auto-start these modes just because the user mentions "brainstorm", "tribunal", "forge", or "review". In the Agon UI, orchestration starts only from explicit slash commands (`/brainstorm`, `/tribunal`, `/forge`, `/review`, etc.) or from explicit CLI/MCP calls. If the user is speaking normally, suggest the slash command instead of starting the workflow.

For non-interactive Codex shell work, the fastest path is the shell bridge:

```bash
agon call brainstorm "What approaches should we consider for this migration?"
agon call tribunal "Should we ship this architecture?" --tribunalMode red-team --rounds 2
agon call forge "Implement the cache layer" --test "npm test"
agon call synthesis "Evolve this design doc into a concrete implementation plan" --swaps 2 --timeout 90
agon call review
```

If `agon` is not linked in the shell, build/link it first:

```bash
npm run build
npm run install:cli
```

Machine-readable callers should add `--jsonl`:

```bash
agon call brainstorm "Compare options for this refactor" --jsonl
```

Use `--cwd <path>` when the target repository is not the current working directory. Use `--engines <comma-separated-registered-ids>` only when the user explicitly requests manual pinning. Use `--timeout <seconds>` for long-running tasks.

External engines should always call Agon through the shell bridge, not direct model CLIs:

```bash
agon call <workflow> "<input>" [flags]
```

Example workflows: `forge`, `brainstorm`, `synthesis`, `tribunal`, `campfire`, `pipeline`, `review`, `goal`, and `team-*`.

When a workflow calls for the full roster, resolve every currently usable, non-excluded engine from the live registry at dispatch time. Do not encode a preferred model, fixed roster, vendor/version order, ratings-based review route, or fallback list in this file. Canonical selection and confidence rules live in `~/.claude/AGENTS.md`.

### Agon Mode Guide

- `brainstorm`: use for open-ended approaches, architecture options, migration strategy, naming/API design, and "what are we missing?" questions.
- `team-brainstorm`: use when idea quality matters enough to have teams synthesize and compete. Good for major product/architecture direction.
- `tribunal`: use for tradeoffs, risky decisions, disagreement, review of a proposed plan, or "argue both sides."
- `team-tribunal`: use when debate quality matters; teams argue positions and a judge synthesizes.
- `forge`: use when multiple engines should implement the same bounded coding task and compete under a fitness command.
- `team-forge`: use for high-value implementation where teams of engines collaborate and compete.
- `synthesis`: use for cross-pollination where engines iteratively improve each other's drafts and a judge selects the best evolved result.
- `campfire`: use when the problem is fuzzy and needs exploratory discussion before a crisp plan exists.
- `pipeline`: full sequence: brainstorm, forge, then tribunal review. Use for critical changes where design, implementation, and review all matter.
- `review`: use for code review of uncommitted changes or a specified target.
- `agent` / `delegate`: use from the interactive app or MCP tools for bounded specialist help, not full competition.
- `QuickNero`: use from MCP/Cesar flows for a fast self-challenge before escalating to heavier modes.
- `plan`: use when the workflow needs staged execution, resumability, approval, or cost visibility.

### Team Modes

Team modes are available through both direct commands and `agon call`:

```bash
agon call brainstorm "Design plugin loading" --team --members 2
agon call team-brainstorm "Design plugin loading" --members 3
agon call forge "Implement session persistence" --team --members 2 --test "npm test"
agon call team-forge "Implement session persistence" --members 3 --test "npm test"
agon call tribunal "Pick the safer rollout plan" --team --members 2 --tribunalMode adversarial
agon call team-tribunal "Pick the safer rollout plan" --members 3 --tribunalMode red-team
```

Interactive equivalents:

```text
/team-brainstorm 2v2 <question>
/team-tribunal 3v3 red-team <question>
/team-forge 2v2 <task> test with <command>
```

### Tribunal And Campfire Options

Tribunal modes:

```bash
agon call tribunal "Question" --tribunalMode adversarial
agon call tribunal "Question" --tribunalMode synthesis
agon call tribunal "Question" --tribunalMode steelman
agon call tribunal "Question" --tribunalMode socratic
agon call tribunal "Question" --tribunalMode red-team
agon call tribunal "Question" --tribunalMode postmortem
```

Campfire strategies:

```bash
agon call campfire "Explore the failure modes" --strategy all-respond

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KERNlang/agon](https://github.com/KERNlang/agon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
