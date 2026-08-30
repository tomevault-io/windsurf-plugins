---
trigger: always_on
description: Instructions for AI agents working on this repository.
---

# AGENTS.md — Way of Work

Instructions for AI agents working on this repository.

## Project Stage — read this first

This repo is **greenfield, pre-v1, MVP state**. There are **no production users, no deployment, and no backward-compatibility commitments**. Nothing here is load-bearing for anyone yet. The goal of every change is to lay a **solid foundation for after v1**, not to protect an installed base that does not exist.

Default posture for any feature or fix:

- **Clean over compatible.** Prefer the correct end-state design over an incremental migration that preserves current internals. There is no old behavior to keep working — if a refactor is right, do it in one move and delete the old path. No deprecation shims, no dual code paths, no feature flags "for safety".
- **Correctness and simplicity first.** Choose the simplest design that is actually correct. Remove special cases instead of accumulating them. Fewer moving parts, fewer producers/owners of a concern, single source of truth.
- **Practical, not speculative.** Build what the current problem needs. Do not add extensibility, config surface, or abstraction layers for hypothetical future requirements ("YAGNI"). Generalize only when a second concrete case exists.
- **No migration-safety scaffolding.** Because nothing is deployed, skip work whose only purpose is staying green across a rollout: before/after parity tests, phased "keep it running between steps" sequencing, compatibility adapters. Phases are still useful as review/checkpoint boundaries; the integration/E2E workflow is the gate (see Workflow §3), not equivalence-to-today.
- **Don't over-engineer.** When a design starts adding layers to hedge risk that only exists for live systems, stop and pick the clean single-path version instead. If you find yourself preserving an awkward boundary "to be safe", that is the signal to simplify.

This posture holds **until this section is explicitly changed**. When v1 ships and real users/deployments exist, revisit these defaults — but do not assume that has happened.

## Workflow

1. **Plan before code when warranted** — New features or PoCs should start with a plan document in `docs/plan/`. Format: `YYYYMMDDNN_<slug>.md`. The plan contains phases, decision log, exit criteria, and out-of-scope items.
   - Bug fixes or isolated changes on top of an existing plan should append to that existing plan instead of creating a new one.
   - Small, focused feature adjustments can skip a new plan file when the scope is obvious and contained.

2. **Phase-based implementation** — Work proceeds one phase at a time:
   - Implement the phase
   - Run self-tests against the phase exit criteria using unit tests or other isolated local verification
   - Proceed to the next phase once the phase passes isolated validation locally

3. **Integration verification** — After all phases are complete:
   - Push the branch to GitHub to trigger the relevant E2E or integration workflow
   - If the required workflow does not trigger automatically, dispatch it manually
   - Choose the workflow to run based on the scope of the change
   - Use the GitHub workflow result as the final verification gate
   - Once the required push checks are green, open a PR against the appropriate base branch

4. **Commit discipline**:
   - One commit per phase (not per file, not per feature)
   - Commit message format: `<type>: <short description>` (e.g. `feat: add mcp approval flow`, `chore: project init`)
   - Never commit secrets, `.env` files, or `node_modules`
   - Push after all phases are complete so GitHub workflows can verify the full change
   - Create the PR only after the required push checks pass

5. **Document decisions** — When making a non-obvious choice (library, pattern, architecture), add it to the active plan's Decision Log table. Future sessions can read this to understand why things are the way they are.

6. **Environment variable discipline** — When adding, renaming, or removing an environment variable, update every required surface in the same change: `docker-compose.yml`, `.env.example`, `README.md` Deployment Configuration, relevant GitHub workflow env blocks, tests/fixtures, and any active plan docs. Do not leave required env vars documented only in code or compose.

7. **Behavior-focused tests** — Prefer tests that prove user-visible behavior, safety boundaries, integration contracts, and non-obvious fail-fast paths. Avoid tests that only lock obvious string construction, env-var trimming/default wrappers, one-line pass-through helpers, or other implementation details unless that exact output is a meaningful product/API contract. If the code is straightforward and already covered through a higher-level behavior test, prefer no direct unit test over low-value coverage.

8. **Rate limiting** — App-level rate limiters / DDoS protection are deferred to infrastructure (ingress, proxy, WAF, or platform controls). CodeQL missing-rate-limit alerts are acknowledged, but do not add Express middleware limiters unless a future plan explicitly changes this policy.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scoutqa-dot-ai/thor](https://github.com/scoutqa-dot-ai/thor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
