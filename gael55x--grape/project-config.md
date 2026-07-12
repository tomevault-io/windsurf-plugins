---
trigger: always_on
description: These rules apply to AI coding agents working on Grape.
---

# Agent Operating Rules

These rules apply to AI coding agents working on Grape.

## Before Editing

Read these files first:

1. `docs/v1/README.md`
2. `docs/v1/SPEC.md`
3. `docs/v1/architecture/invariants.md`
4. `docs/v1/architecture/state-machine.md`
5. `docs/v1/planning/implementation-roadmap.md`
6. The specific domain doc for the module you will edit

If the needed behavior is not documented, update the docs before or with the implementation.

Do not implement from placeholders. If a supporting doc says what it should contain but does not define the contract, read `docs/v1/SPEC.md` and harden the supporting doc first.

Before adding code to an existing large file, read `docs/v1/architecture/overview.md#code-modularity-standards`. If the file is past the split checkpoint, split by ownership before adding another responsibility.

The current in-memory smoke harness is not a substitute for typechecking or behavioral tests. After the In-Memory Context Loop, do not add broad feature code until Project Skeleton And Tooling wires real TypeScript and test gates.

## Documentation Traversal

Use `docs/README.md` for the top-level map and `docs/v1/README.md` for V1 work.

For V1 implementation, start with `docs/v1/SPEC.md`, then read only the folder that owns the change:

- `docs/v1/architecture/` for system shape, state machine, and invariants.
- `docs/v1/core/` for trust, compression, storage, and security.
- `docs/v1/contracts/` for context artifact and context diff schemas.
- `docs/v1/interfaces/` for MCP and CLI contracts.
- `docs/v1/quality/` for tests and benchmarks.
- `docs/v1/planning/` for roadmap, logs, and changelogs.
- `docs/v1/decisions/` for accepted ADRs.
- `docs/v1/examples/` and `docs/v1/fixtures/` for serialized examples and fixture expectations.
- `docs/v1/legacy/alpha/` for historical alpha-era documentation only. Do not treat legacy docs as current implementation input.

Do not add new V1 topic files directly under `docs/v1/` unless they are canonical anchors like `README.md` or `SPEC.md`.

## Documentation Privacy Hygiene

Committed documentation must not include personal local paths, usernames, home directories, private workspace names, local machine cache paths, API keys, tokens, or other machine-specific identifiers. Use neutral placeholders such as `<repo-root>`, `<external-workspace>`, `<temporary-cache>`, or repo-relative paths instead.

Before finishing any documentation edit, scan the changed docs for accidental local identifiers. At minimum, check for home-directory paths, private workspace names, absolute temp/cache paths, and secret-looking values. If command history is worth documenting, rewrite it into reproducible, environment-neutral commands before committing it.

Lesson learned: benchmark notes and trial logs can be useful evidence, but raw local command transcripts often contain personal paths. Preserve the evidence, not the machine-specific details.

## Documentation Style And Release Claims

Future documentation changes must:

- use plain language and short sentences
- avoid hype, vague claims, and zombie nouns when they hide the subject or verb
- avoid unsupported benchmark claims, production-readiness claims, and broad major version readiness claims unless proven
- keep technical terms only when they are accurate (for example: compiler, artifact, context pack, session ledger, claim, proof, retrieval, MCP, CLI, SQLite)
- avoid em dashes, arrow symbols, not equal signs, and emojis
- use plus signs and tildes only when technically required
- verify docs against code before changing release-facing claims
- update stale docs when changing CLI, MCP, package, benchmark, compiler, retrieval, session, ledger, restore, invalidation, claim, proof, or artifact behavior

Zombie noun example:

Bad: `The implementation of retrieval optimization enables better utilization of context.`

Better: `Grape ranks retrieved context before it sends a context pack.`

Benchmark claim policy:

- Docs may say Grape includes benchmark fixtures and scripts for local comparison.
- Docs may not claim proven token reduction percentages, external tool superiority, benchmark-proven savings, or production readiness unless committed benchmark artifacts prove the claim and the docs name fixture, command, date, and limits.
- Label local uncommitted JSON as local fixture results, not official release benchmarks.

Before finishing documentation edits, scan changed docs for banned style and accidental local identifiers.

## Commit Message Hygiene

All agent-created commits must use Conventional Commits format:

```text
<type>(optional-scope): <description>
```

Use the smallest accurate type, such as `docs`, `fix`, `feat`, `test`, `refactor`, `chore`, `build`, or `ci`. If the user asks to commit or push without giving a message, choose a concise Conventional Commit message that describes the actual staged change.

Lesson learned: even documentation-only follow-up commits should follow the repository's commit convention. Do not use ad hoc prose commit messages.

## How To Choose The Right Module

- CLI behavior belongs in `src/cli/`.
- MCP tool transport and schemas belong in `src/mcp/`.
- Workflow orchestration belongs in `src/app/`.
- State transitions belong in `src/core/state/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gael55x/Grape](https://github.com/gael55x/Grape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
