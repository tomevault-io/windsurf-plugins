---
trigger: always_on
description: This file is for implementation work in the wikitool source checkout. `CLAUDE.md` at the repo
---

# Wikitool Development Guidance

This file is for implementation work in the wikitool source checkout. `CLAUDE.md` at the repo
root is a byte-identical mirror of this file; edit both together. It is not the packaged
operator guidance shipped to wiki users. User-facing AI companion guidance lives in
`ai-pack/CLAUDE.md`, `ai-pack/AGENTS.md`, `ai-pack/.claude/skills/`, and
`ai-pack/codex_skills/`.

When changing shipped behavior, update the relevant operator guidance and regenerate
`docs/wikitool/reference.md`. When changing only internal implementation practice, keep the change
scoped here.

## Implementation Rules

- Closely corroborate all implementation work against the authoritative sources for the project:
  specifications, existing code, documentation, tests, and observed runtime behavior.
- Prefer directly evidenced behavior over inferred design.
- When work reveals a canonical or directly evidenced name that supersedes a current label, stage
  that rename across all relevant locations in the same changeset unless a documented blocker
  prevents immediate closeout.
- Implement for correctness first.
- Treat established naming, structure, and subsystem boundaries as evidence, not obligations.
- Preserve them where they aid correctness or comprehension, but not mechanically.
- Where behavior is not directly established, state the uncertainty explicitly, document the gap at
  the relevant site, and do not present hypotheses as facts.
- Do not silently infer missing behavior.
- Do not add defensive code, fallback paths, or error-mitigating logic that obscures divergence from
  the specification or expected behavior.
- Surface errors, mismatches, and unhandled states immediately and locally.
- Prefer explicit assertions, narrow failure points, and observable diagnostics over hidden recovery.
- If the correct behavior at a site is unknown, that unknowing should be visible in the code.
- Write lean, maintainable code with high local comprehensibility.
- Minimize implicit state, cross-file indirection, and abstractions not yet justified by repeated
  evidence.
- Avoid premature generalization.
- Only extract shared machinery when multiple cases demonstrably share the same behavior and
  constraints.
- Use full-cutover judgment where appropriate, but confine changes to what is directly motivated by
  the current work.
- Do not perform speculative rewrites of adjacent code just because it appears improvable.
- If adjacent code is suspect, note it and continue.

## Source Contracts

- Avoid regex-based parsing for wikitext, HTML extraction, and command-contract logic. Use
  deterministic state machines, structured parsers, or character-by-character parsing.
- Keep CLI output contracts explicit. Agent-facing commands should prefer `--format json` when the
  output is consumed programmatically.
- Hidden maintainer commands belong behind the explicit `maintainer` feature; default
  builds are end-user builds.
- The runtime project root is the caller's wiki project, not this source checkout, unless the
  command explicitly accepts a repository root.
- The catalog database at `.wikitool/data/wikitool.db` is disposable. The sync store at
  `.wikitool/sync/sync.sqlite3` is durable revision identity; resets and refreshes must preserve it.

## Verification

- Run targeted unit tests for touched modules.
- Run `cargo test --workspace` before considering source changes complete.
- Run `cargo clippy --workspace --all-targets -- -D warnings` for maintainer-facing cleanup or
  release-adjacent changes.
- For CLI contract changes, run the relevant command help and regenerate
  `docs/wikitool/reference.md` with
  `cargo run --package wikitool --features maintainer -- docs generate-reference`.

## Bounded Output

Contextmink is a separately versioned, project-generic tool. A release bundle
includes its hash-verified upstream pack under `contextmink/`; run
`contextmink/contextmink(.exe) setup-project <project-root> --skill-target both
--json` and follow its receipt-backed setup guidance. In this source checkout,
`bash scripts/fetch_contextmink.sh --platform <platform>` stages the pinned
upstream release under `dist/contextmink-dist/`; it does not install or rebuild
Contextmink. Use the installed Contextmink when a file/text/JSON/SQLite/command
read may produce more output than the transcript should carry.

- Choose invocation by the active shell and target: use `scripts/contextmink ...`
  from Bash-hosted sessions such as macOS, Linux, Git Bash, WSL, or Claude Code;
  use `tools/contextmink/bin/contextmink(.exe) ...` directly from Windows
  PowerShell for contextmink commands; use
  `tools/contextmink/bin/contextmink-bridge.exe --script scripts/contextmink ...`
  when a PowerShell-hosted Windows session needs the Bash launcher or another
  Bash-first repository script.

- Start with `dirs` to orient in an unfamiliar tree, then `files` or `grep`
  for candidate discovery. Prefer `files --ext json` / `--extension jsonl`
  across Windows-to-Bash boundaries because wildcard globs can expand before
  contextmink receives them.

- Read source files through `outline` then `slice`, not dump windows. A named
  file is still reconnaissance while the answer's location inside it is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remiliacorporation/wikitool](https://github.com/remiliacorporation/wikitool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
