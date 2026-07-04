---
trigger: always_on
description: Elixir-native multi-agent AI platform. Phoenix gateway, OTP-supervised agents, Rustler NIFs for crypto/tokenization only.
---

# Fermix

## Project
Elixir-native multi-agent AI platform. Phoenix gateway, OTP-supervised agents, Rustler NIFs for crypto/tokenization only.

**Stack:** Elixir, Phoenix, OTP, Rustler, SQLite
**Predecessor:** RustyClaw — reference for channels, tools, providers

## Behavioral Guidance
- The approved design is the plan. Implement against it, do not quietly re-design the task mid-flight.
- Don't assume. State assumptions explicitly before coding. If multiple interpretations exist, surface them instead of picking silently.
- If the request or design is unclear, stop and ask. If repo reality conflicts with the design, surface the mismatch before coding.
- Prefer the simplest correct solution. No speculative abstractions, no extra flexibility, no "while I'm here" cleverness.
- Make surgical changes. Touch only what the request requires. Mention unrelated issues, don't fix them unless asked.
- For multi-step work, define success in `step -> verify` form and keep going until the checks pass.
- If 200 lines could be 50, rewrite it.

## Execution Contract
- If changing behavior, write or update a failing test first.
- Implement the smallest change that satisfies the design.
- Run the relevant repo commands below before calling the work done. Default expectation: typecheck or build, tests, and lint.
- For docs, config, or scaffolding changes, run the relevant checks and say what is not applicable.
- When a change adds, removes, or materially alters a feature, capability, tool, channel, provider, config surface, or CLI verb, update the `self_knowledge` skill (`apps/fermix_core/priv/skills/self_knowledge/SKILL.md`) in the same change. It is Fermix's runtime self-reference for explaining and fixing itself, and goes stale silently otherwise.
- When adding/altering a **tool, provider, or run-type**, route its telemetry through the shared emitters so it stays correlatable (and Opik-traceable) — tool events via `FermixCore.Tools.Telemetry.exec/5`, provider calls via `FermixCore.Providers.Telemetry.emit_call/3`; **never** hand-roll `:telemetry.execute([:fermix, :tool|provider, ...])`. A new run-type needs a unique `session_id` (+ `parent_session` if spawned) and lifecycle bookend events; a genuinely new event name/run-kind also needs a `FermixOpik` update (the in-umbrella `apps/fermix_opik`). See `docs/TELEMETRY_CONTRACT.md`.
- Never mark work done without proof.

## Code Rules (Non-Negotiable)

1. **Linear flow.** Max 2 nesting levels. Top to bottom.
2. **Bound loops.** Explicit max on retries, polls, recursion. Define cap behavior.
3. **Small functions.** 40-60 lines max. One job per function.
4. **Own resources.** Open → close on every path, including errors.
5. **Narrow state.** No module globals. Pass deps explicitly.
6. **Assert assumptions.** Guards and validation on every public function. Fail loud.
7. **Never swallow errors.** No bare `rescue`. No `{:error, _} -> :ok`. Log, raise, or return.
8. **Visible side effects.** I/O obvious at call site. Separate pure from effectful.
9. **Minimal indirection.** Readable > elegant. One layer of abstraction max.
10. **Surgical changes only.** Touch only what the request requires. Do not refactor adjacent code, comments, or formatting unless the task needs it. Remove only the dead code your change creates.
11. **Warnings = errors.** Linters, typecheckers, analyzers are hard gates. Zero warnings.
12. **No fallbacks.** One code path per behavior. Do not add a "fallback" branch that silently retries with a different mechanism, reads from a deprecated location, or degrades to a partially-working state when the primary path fails. Fallbacks double the surface area, hide which path actually ran, mask real failures behind "it kind of worked," and turn every bug into a five-branch investigation. The old flow is dead the moment the new flow ships — delete it; do not keep it as a safety net. If the primary path fails, fail loud at the boundary with a clear message and exit non-zero. Two valid configurations are fine (e.g., user-scope vs system-scope service); two paths to handle one configuration is not. If you think you need a fallback, you actually need (a) a clearer error message, (b) a single failure-recovery step for a destructive op (e.g., upgrade rollback — explicitly scoped, no user-facing chain), or (c) a different design that doesn't have the failure mode at all.

## Conventions
- `@callback` for all plugin interfaces (providers, channels, tools)
- `{:ok, result} | {:error, reason}` tuples, not exceptions
- GenServer callbacks thin — delegate to private functions
- No business logic in Phoenix controllers
- Typespecs on all public functions

## Commands
```sh
mix deps.get && mix compile
mix test
mix credo --strict
mix format --check-formatted
```

## Docs
- `docs/TELEMETRY_CONTRACT.md` — Telemetry/observability contract — how new tools/providers/run-types stay correlatable + Opik-traceable (shared emitters, `session_id`/`parent_session`, content gating, the `fermix_opik` rule)
- `docs/PROJECT_PLAN.md` — Full plan with phases
- `docs/PHASE1_TASKS.md` — 16 tasks with implementation code
- `docs/ROADMAP.md` — Post-MVP feature roadmap (M2-M9)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tezra-io/fermix](https://github.com/tezra-io/fermix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
