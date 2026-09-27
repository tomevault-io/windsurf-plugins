---
trigger: always_on
description: <!-- kb:context scopes/repository--cdb4ee2aea69 -->
---

<!-- kb:context scopes/repository--cdb4ee2aea69 -->
# Contents

- `src/` – deterministic Markdown graph and attachment analysis, typed metadata and exact repository-scope queries, local hybrid retrieval, bounded Git provenance, code-mode sessions and DAG workflows, frozen-corpus evaluation authoring and execution, safe single-note authoring, percolation, repository-memory routing and audits, the advisory source inbox, structural navigation, static `hraness.wordcell.site.v1` publication, initialization, CLI, capture, URL intelligence, and diagnostic code with colocated tests.
- `src/workflows/` – reusable code-mode decision-context, change-explanation, and plan-radar workflows with bounded parallel execution.
- `dist/` – committed Bun-targeted ESM entrypoints plus the compiled Defuddle worker and the browser-targeted publish reader bundle.
- `skills/wordcell/` – the single public Agent Skill for querying, capturing into, planning in, percolating, refreshing, and validating a hraness/wordcell vault, with focused workflow references loaded on demand.
- `.agents/skills/` – internal plan authoring, phased execution, implementation, and independent review workflows.
- `kb/` – this source repository's authored rationale, maintained synthesis, and implementation plans; it is separate from the package's graph implementation and fixtures.
- `WRITING.md` and `STYLE.md` – internal and public prose contracts.
- `docs/` – design, capture, and agent-workflow documentation.
- `site/` – the wordcell.io Next.js site: marketing/docs pages plus the hosted publication API under `app/api/v1/` (capability tokens, bounded vault intake, server-side projection, digest-addressed artifact storage) and the `lib/hosted/` server modules.
- `worker/` – the `wordcell-sites` Cloudflare Worker holding the R2 bucket binding: signed-object operations for the API and the public `/p/<key8>/<slug>/` read path.
- `.github/workflows/` – read-only branch validation, canonical attested immutable GitHub Releases with automatic npm publication of the same bytes, and the wordcell.io site build.
- `portfolio-inventory.json`, `scripts/check-portfolio-inventory.ts`, and `scripts/check-installed-command-docs.ts` – canonical public package inventory and standalone public-command consistency gates.
- `README.md`, `CONTRIBUTING.md`, `SECURITY.md`, and `LICENSE` – public usage, project policy, threat model, and terms.
- `package.json`, `tsconfig.json`, and `bun.lock` – standalone package and frozen verification configuration.

# Search repository knowledge

Use `bun run kb:search "question" --json` for ordinary questions about this
repository's `kb/`. It pins Wordcell 0.22.3 and enables hosted TypeSafe
reranking for this public KB. Queries and bounded note identifiers, titles,
paths, and snippets leave the machine. Keep the API key in the private
Wordcell credential file or environment, never in this repository.

Inspect the rerank lane before claiming it ran: `ready` means the complete
window was accepted; `unavailable` or `degraded` preserves baseline results.
Use `bun run kb:search:local "question" --json` for local-only search. Read the
returned notes and source links before changing code; ranking is not proof.
Existing catalog, context, and maintenance commands retain their own roles.

# Guidelines

- Use Bun 1.3.14 for repository commands and keep the authored Markdown compatible with Obsidian and ordinary text tooling.
- Follow `WRITING.md` for internal prose and `STYLE.md` for public prose.
- Follow the shared [Hraness README guidelines](https://github.com/hraness/.github/blob/main/README_GUIDELINES.md) for the README trust path and its website projection. Adapt the structure to Wordcell's local-data and Agent Skill boundaries instead of copying a fixed template.
- Apply unreasonably robust programming when agent work is cheap. Model invalid states out of existence, parse every foreign value from `unknown`, and pair readable deterministic regressions with property tests for parsing, resolution, ordering, path confinement, and round trips.
- Treat a stable package or release version as exactly three canonical decimal components in the inclusive range `0..Number.MAX_SAFE_INTEGER`. Reject larger components at selection, package preparation, artifact verification, attestation verification, and release ordering boundaries.
- Deliver changes to `main` through a current-head pull request. Keep the stable `Required` CI job green, resolve every review thread, and serialize merges. Human approval stays optional while one regular maintainer would otherwise self-review. Never force-push or bypass the gate.
- Pin Hraness dependencies to reviewed immutable releases or full commits. Never connect repositories through sibling paths, Git submodules, or coordinated `main` assumptions; upgrade each consumer independently.
- Extract a shared package only after two concrete consumers require the same stable interface. Keep every shared package product-neutral and free of product imports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hraness/wordcell](https://github.com/hraness/wordcell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
