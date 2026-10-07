---
trigger: always_on
description: This is the brief every reviewer of a pull request in this repository follows — GitHub Copilot, an agent, or a human. The repository's hard rules (formatting, imports, type discipline, docstring conventions, dead code, line budgets, coverage, AI-attribution) are enforced mechanically by the CI gates and are documented in `CLAUDE.md`, the constitution. Those are not your job to re-check. This guideline is for the semantic review the gates cannot do: whether the change actually does what it claims
---

# PR review guideline

This is the brief every reviewer of a pull request in this repository follows — GitHub Copilot, an agent, or a human. The repository's hard rules (formatting, imports, type discipline, docstring conventions, dead code, line budgets, coverage, AI-attribution) are enforced mechanically by the CI gates and are documented in `CLAUDE.md`, the constitution. Those are not your job to re-check. This guideline is for the semantic review the gates cannot do: whether the change actually does what it claims, and what it breaks that is not in the diff.

Review like a senior engineer, not a linter. Be meticulous, terse, concrete, and honest. Praise carries no information; cut it. The opinion is the deliverable. When review posting is authorized, place findings inline on the exact file and line. Otherwise prepare the review locally; this guideline does not authorize external messages.

## 0. Before anything else: the attribution stop (BLOCKING)

Scan every commit message between the base and `HEAD`, and the PR title and body, for any mention of an AI or LLM assistant: `claude`, `codex`, `chatgpt`, `gpt-<n>`, `copilot`, `cursor`, `gemini`, `anthropic`, `openai`, `llm`, `ai-assistant`, the phrase "generated with", and any `Co-Authored-By:` trailer naming an assistant. A topical reference to a file literally named `copilot-instructions.md`, or to the repository's required "Copilot review", is not attribution.

Any hit is a blocking failure. Stop and report only that. The author must rewrite history to strip the mention(s) before the PR can move forward. PRs in this repository never carry assistant attribution — this is non-negotiable, and the `pr_checks_cc` gate enforces it server-side, so an unstripped mention will block the merge regardless.

## 1. Read before you judge

Read the PR description and any linked issue first. Know what problem the change solves before you read the diff. Then sit with the diff — understand the change before flagging anything. The difference between genuine help and noise dressed as help is whether you understood the code before judging it.

## 2. Look beyond the diff

This is the core move. The diff shows what the author touched; it does not show what the author forgot. The bug is rarely in the lines that changed — it is in the line three files away that still expects the old shape.

- For every signature change, find every caller.
- For every renamed field, grep every consumer, config file, and doc.
- For every new enum or variant, find every `switch` / `match` / `if`-chain on that type and confirm it handles the new case.
- For every changed wire format or serialized shape, find the other side.

Treat the diff as the starting point for a search, not as the territory.

## 3. What to hunt

- **Correctness** — does the logic do what it claims? Edge cases, off-by-one, null/empty handling.
- **Environmental assumptions** — for each changed function, name what it assumes about disk layout, API shape, network state, and config values; for each assumption, ask what breaks when it is wrong.
- **Identity and namespaces** — if a changed value feeds a dedup key, idempotency token, canonical ID, or partition offset, follow it to every consumer and confirm the new shape cannot collide with other values in the same namespace.
- **Stateful loops** — if a loop has an idempotency guard, mentally execute the second run; every branch that persists data must also update the guard.
- **Destructive operations** — for any `delete` / `reset` / `overwrite` / `truncate`, enumerate every path that reaches it, state the invariant, and verify every exit honors it. Distrust string variables used as control-flow guards.
- **Security surfaces** — for auth, permissions, feature flags, and error handling, trace control flow from the entry point and identify every bypass or short-circuit. Ask: what ships if every TODO is never resolved? If a flag-off path is a security bypass, flag the bypass, not the guarded code.
- **Silent failures** — `try`/`except` that swallows, defaults that mask bugs. The fail-loud gate catches the AST-detectable forms; you catch the semantic ones.
- **Component seams** — across any boundary the PR introduces or changes, trace state flow: who owns mutation, what ordering is assumed, what breaks if one side changes independently.
- **Tests** — present and meaningful. A test that does not exercise the thing it names is worse than no test, because it reads as covered.
- **Cruft** — premature abstraction, speculative flexibility, unused generality, dead code, stale comments, leftover debug output. And bad names — especially bad names.
- **Empirical claims** — a benchmark, throughput, or latency claim with no automated gate to catch its regression is a gap; name it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autonomio/talos](https://github.com/autonomio/talos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
