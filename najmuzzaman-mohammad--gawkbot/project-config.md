---
trigger: always_on
description: These instructions apply to AI coding agents working in Nex repositories. Keep
---

# Agent Instructions

## Base Agent Instructions

These instructions apply to AI coding agents working in Nex repositories. Keep
tool-specific root files (`CLAUDE.md`, `AGENTS.md`, etc.) pointed at the same
canonical repo file so Claude, Codex, and other agents receive equivalent
guidance.

No additional setup is required beyond a normal clone of the repository. The
`.github` repository provides rollout tooling and templates only; committed
repo-local instruction files are the runtime source of truth for contributors.

### Working Rules

- Read the relevant code before editing. Do not reason from assumptions when
  the repository can answer the question.
- Prefer narrow, surgical changes that follow existing repo patterns.
- Do not revert changes you did not make unless the user explicitly asks.
- Own bugs surfaced during the work. Do not dismiss them as unrelated when they
  block the requested outcome.
- Ask before destructive or hard-to-reverse actions: deleting state, clearing
  Docker volumes, applying migrations outside local dev, or changing production
  infrastructure.

### Git And PRs

- Never push directly to `main` (repos with other contributors; see each repo's
  own profile below for exceptions).
- Use a branch and open a draft PR for code changes.
- Use Conventional Commits for commit messages.
- Run the repo's documented checks before opening or marking a PR ready.
- Do not use `--no-verify` to bypass hooks.

### Quality Bar

- Do not suppress lint or type errors with ignore comments. Fix the code.
- Do not introduce explicit `any` in TypeScript. Use specific types, `unknown`
  with narrowing, or preserve existing untyped boundaries.
- Do not commit secrets, tokens, credentials, or inline API keys.
- Source required secrets from environment files, secret managers, or the
  repo-documented local setup.
- Treat E2E failures as product signals. Do not hand-wave them away.

### Nex Context

Nex is a context graph platform for AI agents, not a CRM. Do not describe the
product as a CRM in code, comments, docs, or external copy.

Use the available Nex memory/context tools when they are installed:

- Query context for people, companies, projects, or prior decisions when that
  context would materially improve the answer.
- Store durable user preferences, project decisions, and lessons learned when
  the user asks to remember them or when future sessions clearly need them.
- Scan repo docs and instruction files after meaningful updates so project
  context stays discoverable.

Tool names differ by platform. Use the equivalent available surface, for
example `query_context` / `nex_ask`, `add_context` / `nex_remember`,
`scan_files`, or `ingest_context_files`.

### Triangulation through orthogonal sub-agents

Use this pattern for high-stakes design decisions, including security
boundaries, wire shapes, schema changes, and new public API surfaces.

1. **Don't trust a single agent's review.** Even with a thorough prompt, one
   agent has one frame.
2. **Spawn 3-5 sub-agents in parallel**, each with a different lens preamble:
   security, perf, API, SRE, architecture, types, or distributed systems. Use
   `bash scripts/dispatch-triangulation.sh`.
3. **Aggregate their outputs.** Findings that 2+ agents flag independently are
   high-confidence. Singletons are lower confidence; verify before fixing.
4. **Direct disagreements** are signals to escalate to human review, not to
   pick a side.
5. **Use this pattern especially when:** introducing a new wire shape; changing
   a security-relevant invariant; designing a new public API; choosing between
   two architectural approaches.

### Verification agents as sounding boards

Use this pattern when Claude, Codex, or a human has a proposed solution and
wants to stress-test it before committing.

1. **Run a verification agent** with
   `bash scripts/dispatch-verification-agent.sh`. Pass the solution, target
   files, and an optional adversarial lens.
2. **The verification agent runs in read-only mode.** It cannot edit; it can
   only find what the solution does not cover.
3. **Treat its findings as a pre-commit review.** Fix what's real, skip with
   reason what's not, and defer what's out of scope.
4. **Use this pattern especially when:** the change is irreversible, such as
   deleting state or dropping schema; the change is in a security boundary,
   such as validators, sanitizers, or freeze boundaries; the change is in code
   with no consumers yet, where downstream would not catch a regression.

### When to use which

| Situation | Pattern |
|---|---|
| Initial design of a new surface | Triangulation (orthogonal lenses) |
| Stress-testing a proposed fix | Verification agent (one adversarial lens) |
| Post-implementation review | Both — triangulation first, then verification on the synthesis |
| Routine bug fix | Neither (overkill) |
| Pre-merge gate | Verification agent + the existing demo + package-specific cross-language oracle (for example, `testdata/verifier-reference.go` in protocol-grade packages) |

## Wuphf Agent Instructions

Use this profile for the Wuphf public repo and its worktrees.

### Commands

```bash
go build -o wuphf ./cmd/wuphf
bash scripts/test-go.sh
bash scripts/test-go.sh ./internal/team
bash scripts/test-web.sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [najmuzzaman-mohammad/gawkbot](https://github.com/najmuzzaman-mohammad/gawkbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
