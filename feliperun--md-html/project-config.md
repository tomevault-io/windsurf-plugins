---
trigger: always_on
description: > [Architecture](docs/ARCHITECTURE.md) · [Abstractions](docs/ABSTRACTIONS.md) · [Vision](docs/VISION.md) · [Getting Started](docs/GETTING-STARTED.md) · [ADRs](docs/adr/README.md) · [Sentrux](docs/sentrux.md)
---

# AGENTS.md — md.html

> [Architecture](docs/ARCHITECTURE.md) · [Abstractions](docs/ABSTRACTIONS.md) · [Vision](docs/VISION.md) · [Getting Started](docs/GETTING-STARTED.md) · [ADRs](docs/adr/README.md) · [Sentrux](docs/sentrux.md)

Write the minimum code that runs. No fluff, no gold-plating.

- Do not preserve backward compatibility. Remove obsolete paths instead of adding
  compatibility layers, fallbacks, or migrations.
- Choose the simplest implementation that fully meets the current requirements.
  Avoid speculative abstractions, configuration, and indirection.
- Grow the system in layers. Start from the smallest version that works end to end,
  and add each new capability on top of a product that already works. Never trade a
  working product for unfinished complexity.
- Keep components modular and concerns clearly separated.
- Prefer established, well-maintained libraries when they reduce overall complexity
  or improve reliability. Do not reimplement common functionality without a clear reason.
- Lean on the dependencies already in the project before writing your own
  implementation or adding packages. Do not assume a library lacks a capability
  without checking its documentation and types.
- Make architectural decisions for the long term. Do not accept a stopgap that only
  works for now and is meant to be replaced later.
- Study how established products solve the problem before designing a solution. Adopt
  their proven patterns and conventions rather than inventing an approach from scratch.

## Repository rules

- **`AGENTS.md` is the single source of guidance.** `CLAUDE.md`, `GEMINI.md`,
  `CURSOR.md`, `AGENT.md` and `.github/copilot-instructions.md` are symlinks to it.
  Never edit a symlink; never let one drift into a real file.
- **Never commit secrets.** Tokens, credentials, and service-account JSON stay in a
  secret manager or a gitignored `.env`. The `pre-commit` hook scans the staged diff;
  do not work around it.
- **No personal or production-derived data in source**, migrations, fixtures, tests,
  or docs. Committed fixtures are synthetic. User-specific values belong in runtime
  configuration.
- **Never expose internals to users.** No stack traces, internal URLs, or env var
  names in user-facing copy.
- **Conventional Commits required.** `feat:`, `fix:`, `docs:`, `refactor:`, `test:`,
  `chore:`. One logical change per commit; one bounded scope per PR. Release tooling
  parses them — a break in a published contract ships as `feat!:` or carries a
  `BREAKING CHANGE:` footer, never as a plain `feat:`.
- **Never `--no-verify`.** If a hook blocks, fix the underlying issue.
- **Code, comments, and identifiers in English.** Surgical changes — no opportunistic
  refactors in feature PRs, no suppression comments to silence a linter.
- **Shell scripts** run under `set -euo pipefail` and are idempotent — re-running
  completes what is missing instead of duplicating or destroying.

## Workflow

- Check `docs/adr/` before any structural choice. Branch from `main`.
- **TDD for behavior changes**: red → green → refactor → commit. Bug fixes start with
  a failing regression test. Exception: pure docs, formatting, or copy changes.
- **E2E for key features**: any user-visible change to a primary workflow adds or
  updates a deterministic E2E scenario, isolated from real data and credentials.
  Unit tests do not replace it.
- **ADRs** live in `docs/adr/`, one decision per file, created in the same commit as
  the code (`/create-adr`). Never edit an active ADR — supersede it. Required for a
  new dependency that changes surface area, a storage or schema convention, a core
  abstraction, a hosting or secrets strategy, or a cross-cutting pattern. Not for
  behavior-preserving fixes, refactors, version bumps, or copy tweaks. After a
  structural change, update `docs/ARCHITECTURE.md` in the same commit — it reflects
  **active** decisions only.
- **Destructive actions** — merging, force-pushing, changing repository permissions,
  dropping schema, deleting data — require explicit human sign-off in the moment.
  An agent that hits this gate hands off to the user rather than routing around it.

## Gates

```bash
# Stack a nascer — quando runtime/ e crates/ existirem, o suíte fica:
#   runtime:  node runtime/build.mjs check  (fixtures .md → HTML esperado)
#   cli:      cargo test                     (gramática, validação, round-trip)
# Enquanto não existe, o placeholder abaixo mantém o gate verde de propósito.
echo 'TODO: define the check suite (typecheck + test)'           # types + tests
sentrux check .           # absolute limits (.sentrux/rules.toml)
sentrux gate .            # no structural regression vs .sentrux/baseline.json
```

CI mirrors this (`.github/workflows/quality.yml`). Before touching existing files run
`sentrux gate --save .` to capture the baseline; before committing run `sentrux gate .`
— degradation on a touched file means refactor, not commit. New files pass
`sentrux check .` clean. **Never silence a rule to pass** — the gate is a ratchet, and
every file you touch leaves with an equal-or-better score.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feliperun/md.html](https://github.com/feliperun/md.html) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
