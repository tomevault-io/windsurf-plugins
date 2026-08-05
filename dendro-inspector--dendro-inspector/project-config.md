---
trigger: always_on
description: - **Owner:** Dendro Inspector maintainers
---

# Agent Guidelines for Dendro Inspector

- **Status:** Draft
- **Owner:** Dendro Inspector maintainers
- **Date:** 2026-07-27
- **Last-verified:** 2026-07-28

**This is a public open-source repository.** Everything committed here is permanent,
world-readable, and mirrored by third parties within minutes. Sections 13-15 carry the
rules that exist *only* because of that; they override anything softer above them.

Distilled from the Azure Subscription Vending - EA repository (`AGENTS.md` §§0-14,
`FABLE.md`, `skills/hard-problem-method`, `skills/verify-before-report`,
`docs/DOCUMENTATION-CREATION-PROCESS.md`, `.githooks/pre-commit.ps1`, commit history).
Project-specific Azure/vending content was dropped; the enforcement shape was kept.

---

## The Main Rule

**Every rule and every contract has exactly one canonical home, and each one is backed
by a gate that can fail. A claim without evidence is a hypothesis — label it as one.**

Everything below is that rule applied to a surface: instructions, code, docs, commits,
status reports. Two sources of truth is the bug this file exists to prevent.

---

## 0. Instruction Boundary

`AGENTS.md` is the single source of truth for agent behavior and project instructions.
Tool entrypoints (`CLAUDE.md`, `.github/copilot-instructions.md`, `AGENTS.md` mirrors)
are bootstrap pointers and **must not** duplicate operational rules. If a rule changes,
change it here.

Corollary: do not add a rule to this file that you cannot point at a gate, a reviewer,
or an explicit owner decision for. Unenforced prose decays into folklore.

---

## 1. Operating Principles (How to Work Here)

1. **Think before coding.** State assumptions explicitly. When a task is ambiguous,
   present the interpretations before editing. Verify against the actual tree
   (`git ls-files`, file reads) — session memory and status docs go stale.
2. **Simplicity first.** The minimum change that solves the stated problem. No
   speculative features, no premature abstraction.
3. **Surgical changes.** Touch only what the task requires. Preserve existing style,
   comments, and load-bearing fallbacks. Never reformat a whole file — a line-ending
   flip buries a 4-line change in a 700-line diff.
4. **Goal-driven execution.** Define verifiable success criteria before editing, run the
   matching gates, and report a status from §3 — never "done" without evidence.
5. **Stale-knowledge rule.** Re-verify any fact sourced from memory, a review doc, or an
   earlier message by live read or grep before acting on it or repeating it.
6. **Do not loop.** Two or three failed attempts on one approach means stop, re-read the
   evidence, re-hypothesize. For incidents: write down ≥3 candidate causes, then pick the
   cheapest test that *discriminates* between the top two — not the one that confirms
   your favorite. Never fix on an unconfirmed hypothesis.

---

## 2. Authorization Boundaries

Operations with real-world side effects require explicit owner approval **in the current
session**; approval in a previous session does not carry over:

- Deploying, releasing, or triggering CI/CD pipelines.
- Cloud CLI / control-plane / data-plane mutations.
- Anything sending data to an external service.
- `git push`, force-push, and branch deletion.

Public-repo additions — these are irreversible in practice, because the internet keeps a
copy:

- Pushing to a public branch, opening or commenting on a public issue or PR, publishing a
  release or tag, publishing a package to any registry (npm, PyPI, crates.io, …).
- Any action that speaks in the project's voice to people outside the maintainer group.
- Adding, changing, or removing a dependency (§14).

Never run `git clean`, `git reset --hard`, or broad restore commands. Never delete
untracked files unless explicitly asked. If verification requires an operation you are
not authorized for, report `BLOCKED` with the exact command for the owner to run.

---

## 3. Evidence-Gated Reporting (Mandatory Vocabulary)

Optimistic reporting is the top failure mode of agent sessions. Use these statuses:

| Status | Meaning | Requires |
|---|---|---|
| `VERIFIED` | claim plus evidence artifact | command output, run ID, query result, or diff |
| `IMPLEMENTED-UNVERIFIED` | edit made, validation not run | name the missing validation and why it did not run |
| `UNKNOWN` | no evidence either way | name what would produce evidence |
| `BLOCKED` | verification needs access you lack | the exact command for the owner |

Forbidden without evidence: *done, works, should work, successfully, wired, fixed,
everything passes*.

Report format:

```text
Claim:
Status: VERIFIED | IMPLEMENTED-UNVERIFIED | UNKNOWN | BLOCKED
Evidence: <command + result | file:line | run id>
Not verified:
Next verification step:
```

Anti-patterns: running a syntax gate and claiming semantic safety; citing a review doc,
a memory entry, or your own earlier message as evidence for current code state;
retro-fitting evidence to a conclusion already written; "tested the happy path" reported
as `VERIFIED` for error handling.

---

## 4. Hard Requirements

Each hard requirement follows one shape — copy it when adding a new one:

```markdown
### N.M <Rule name>
<File or component> is the **source of truth** for <contract>.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dendro-Inspector/Dendro-Inspector](https://github.com/Dendro-Inspector/Dendro-Inspector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
