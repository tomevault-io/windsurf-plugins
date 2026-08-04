---
trigger: always_on
description: Guidance for coding agents working in this repo.
---

# AGENTS.md

Guidance for coding agents working in this repo.

## Repo Shape

- `apps/cli/` - Go CLI source for `newsjack`. Commands live in `apps/cli/cmd/newsjack/`.
- `apps/site/` - Next.js site and distribution build. `proxy.ts` routes curl/wget to `install.sh`; browser traffic gets the site.
- `bin/newsjack` - source-checkout shim that runs the Go CLI with `go run`. Working in this repo, prefer this shim; `~/.newsjack/bin/newsjack` is the end-user install path that public skills reference.
- `skills/` - public Newsjack skills. These are user-facing runtime instructions.
- `fixtures/` - local fixtures and smoke harnesses. Fixture-specific docs, scripts, prompts, and generated run folders belong here.
- `eval/` - evaluation artifacts, including reverse-newsjack methodology and dated run results.
- `harness/` - local agent/runtime harness support.
- `docs/` - planning and architecture notes.

## Canonical Boundaries

- Public skill behavior belongs in `skills/*/SKILL.md` and skill-local support files.
- Fixture usage belongs in the relevant `fixtures/*/README.md`, scripts, or prompt files, not in public skills.
- `skills/newsjack-detector/SKILL.md` is the canonical detector pipeline contract.
- `fixtures/newsjack-detector-agent/PROMPT.md` should only point agents to the canonical detector skill and the fixture profile files. Do not duplicate run commands or profile lists there.
- `fixtures/newsjack-detector-agent/README.md` owns fixture script usage.

## Minimal Memory

Keep Newsjack as stateless as practical. Durable state creates hidden baggage on users' machines, makes behavior harder to reason about, and slows iteration.

- Prefer explicit user-owned files, command inputs, and per-run artifacts over background memory.
- If something can be regenerated with one prompt or one cheap deterministic command, do not save it as durable state.
- If an LLM can derive something from live evidence in one prompt, prefer keeping that derivation in the skill/prompt layer instead of encoding a brittle approximation in Go.
- Persist only state that has a clear product owner and lifecycle: monitor profiles, client briefs, schedules, credentials, run provenance, and recurring-run seen/suppression data.
- Do not persist model-inferred topics, learned assumptions, ranking preferences, or other agent guesses unless the user explicitly promotes them into a profile or brief.
- When state is necessary, make it inspectable, portable, and easy to delete. Avoid adding new local databases, caches, or hidden files unless they are clearly justified.
- One-off commands should be as stateless as possible. Recurring monitors may use limited state for hygiene, but that state should not become the source of truth for discovery or judgment.

## Discovery Doctrine

Open discovery, strict surfacing.

- News discovery must be evidence-first, not model-prior-first. Runtime judgment should rely on collected evidence and explicit profile setup, not remembered sector trends.
- Static profile/query terms should come from explicit user input, the client's own materials, named entities such as competitors/products/regulators, or fresh current coverage. Do not persist model-inferred "hot topics" as durable search terms.
- Setup owns durable discovery aperture: a focused set of short broad beat topics, static search terms, competitors, and feeds belong in the monitor profile, not in per-run generated terms. Prefer 6-8 core topics, usually 2-3 words; one-word topics are fine when they name a real beat.
- Runtime detector skills should not invent or store retrieval terms from model memory or today's headlines. If intake is too narrow or too broad, edit the profile setup file instead of adding hidden run-specific discovery state.
- Preserve the separation between intake and judgment: widen discovery when needed, but keep freshness, story-origin, standing, brand-safety, and angle gates strict before surfacing anything as actionable.

## CLI Unix Principle

Keep the Go CLI a deterministic, composable data layer. Before adding behavior to `apps/cli`, ask whether every Newsjack user needs the same testable operation regardless of client, voice, outlet, or prompt strategy. If the answer is no, put it in a skill, fixture, or user-provided renderer instead.

CLI-owned behavior:

- Install/update/runtime/auth/MCP plumbing, local paths, local state, and health checks.
- Evidence ingestion, normalization, dedupe, clustering, source diagnostics, and local seen-store behavior.
- Mechanical scores, routing metadata, hygiene/safety flags, and deterministic thresholds used to order work.
- JSON-to-JSON transforms such as applying decisions, validating artifact contracts, attaching provenance, and computing freshness status codes.
- Machine-readable facts needed by skills to render honestly: URLs, timestamps, source provenance, status codes, counts, exclusions, and diagnostics.

Skill- or user-owned behavior:

- PR judgment, client standing, newsworthiness, brand-safety judgment, angle fit, journalist shape, handoff decisions, and final labels/framing.
- Report structure, Markdown/prose rendering, story hierarchy, "top news" wording, what to emphasize, and how to explain statuses to a human.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elvisun/newsjack](https://github.com/elvisun/newsjack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
