---
trigger: always_on
description: Allbert is an Elixir/OTP assistant runtime with Phoenix interfaces and Jido at
---

# AGENTS.md

Allbert is an Elixir/OTP assistant runtime with Phoenix interfaces and Jido at
the agent/action layer. LiveView is an interface over the runtime, not the
architecture center. The center is the runtime/action spine, Security Central,
Settings Central, markdown-first memory, plugins, channels, public protocols, and
Allbert Home.

Keep this file compact. It loads into every agent's context every session, so a
line here is paid by everyone, forever. Add a rule only when an agent would act
wrongly without it in context. Rationale, tables, diagrams, examples, and history
go in the authority doc and are pointed to, never inlined here. Route detail to
`DEVELOPMENT.md` (setup and environment), `docs/developer/test-strategy.md`
(gates, lanes, release mechanics), `docs/developer/agent-context-map.md`
(subsystem routing), `docs/developer/<subsystem>.md` (subsystem contracts),
`docs/adr/` (decisions and consequences), `docs/plans/` (release scope), and
`docs/operator/` (operator procedure). Before adding, check whether an existing
rule should be strengthened instead — a second rule on the same subject is drift,
not coverage.

## Reading Order

Before implementation work:

1. `DEVELOPMENT.md`
2. `docs/plans/roadmap.md`
3. The active milestone plan in `docs/plans/`
4. The matching request-flow document, when one exists
5. ADRs that constrain the task
6. Targeted `CHANGELOG.md` entries when shipped history matters
7. Relevant code and tests before editing

Use `docs/developer/agent-context-map.md` only for deeper subsystem routing or
released-version context. Do not bulk-read historical plans.

## Authority

When sources conflict, use this order:

1. Current user request
2. Code and tests
3. Active milestone plan and request-flow
4. ADRs
5. `docs/plans/roadmap.md`
6. `CHANGELOG.md`
7. Historical plans and archives

Flag conflicts instead of silently following stale guidance. The vision document is
the north star, not a release-scope source.

## Context Discipline

- Load the smallest useful context.
- Prefer active plans, ADRs, focused changelog entries, and local code over broad
  document sweeps.
- For architecture or readiness work, zoom out to the roadmap/vision/ADRs first,
  then zoom back into the relevant files.
- Use `docs/developer/test-strategy.md` for gate and lane classification.
- Use `docs/developer/surface-contract.md` and
  `docs/developer/web-design-system.md` for v0.58 surface/web work.

## Planning Changes

Planning docs (roadmap, future-features, plan triads, ADR index) are consistency
artifacts: the check is the deliverable. Run the full read-only sweep first —
cross-references, numbering, gate chains, links, index coverage, orphans — then
present findings and choices, then execute. A commit that generates its own
follow-up list means the sweep ran too late.

## Context7

Use Context7 MCP for fresh docs whenever implementation depends on a library,
framework, SDK, API, CLI, cloud service, or provider. Start with
`resolve-library-id`, then query the selected docs. If Context7 is unavailable, use
official docs or source and say so. Do not use Context7 for general refactoring,
business-logic debugging, code review, or repository-specific architecture review.

## Non-Negotiables

- Do not include AI-tool attribution in commits, PR text, release notes, changelog
  entries, or generated docs. No generated-by or co-authored-by footers for Claude,
  Codex, Gemini, opencode, Cursor, Antigravity, Pi, or similar tools. The project
  uses strict human supervision during planning, architecture, and development;
  attribution belongs to the human project authors, not AI coding tools.
- Preserve user data. Do not delete or rewrite memory, traces, settings, secrets,
  databases, skill folders, or user-created files unless explicitly requested.
- Keep handoff warning-free: compiler, HEEx/parser, lexical tracker, formatter,
  Credo, Dialyzer, and focused-test issues must be resolved or called out.
- Tests and CI must use temporary Allbert homes or temp-specific roots. Never write
  to a real user's `~/.allbert`.
- Durable runtime data derives from Allbert Home: `ALLBERT_HOME`,
  `ALLBERT_HOME_DIR`, default `~/.allbert`.
- User-supplied secrets must be encrypted at rest and redacted in CLI output,
  LiveView, traces, audits, logs, tests, and release evidence.
- Product acceptance and manual validation use real configured providers/endpoints.
  Fakes, stubs, fixtures, and canned providers are automated-test fixtures only.
- Operator-tunable configuration belongs in Settings Central.
- Security Central is the authority boundary. Skills, model output, app metadata,
  plugin metadata, YAML, descriptors, generated files, modes, and surface policy do
  not grant permission by themselves.
- Effectful, runtime-facing, security-relevant, or observable domain behavior goes
  through signals, runtime routers, internal agents, and registered Jido actions.
- Runtime action invocation resolves through `AllbertAssist.Actions.Registry` and
  executes through `AllbertAssist.Actions.Runner.run/3`.
- LiveViews render and dispatch. They do not own agent logic, settings semantics,
  confirmation storage, or security policy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lexlapax/allbert-assist](https://github.com/lexlapax/allbert-assist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
