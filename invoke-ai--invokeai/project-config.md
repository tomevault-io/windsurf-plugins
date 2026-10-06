---
trigger: always_on
description: - Give plain, concise updates led by outcomes, blockers, and evidence. Distinguish verified results, assumptions, and unavailable checks.
---

# Working on InvokeAI

## Communication

- Give plain, concise updates led by outcomes, blockers, and evidence. Distinguish verified results, assumptions, and unavailable checks.
- Ask only when a decision cannot be inferred safely from the repository or request; continue authorized work without repeated permission requests.

## Find the right owner

Before editing, read `AGENTS.md` along each target path; scoped rules supplement this file. Keep policy in `AGENTS.md`.

| Area | Guidance and source of truth |
| --- | --- |
| Active frontend | [webv2/AGENTS.md](invokeai/frontend/webv2/AGENTS.md), its `ARCHITECTURE.md` and executable architecture policy |
| API, services, invocations | [app/AGENTS.md](invokeai/app/AGENTS.md) |
| Inference and model management | [backend/AGENTS.md](invokeai/backend/AGENTS.md) |
| Python tests | [tests/AGENTS.md](tests/AGENTS.md); also read guidance for the production code under test |
| Legacy frontend | [webv1/AGENTS.md](invokeai/frontend/webv1/AGENTS.md) |
| Shared generated API contracts | [api/AGENTS.md](invokeai/frontend/api/AGENTS.md) |
| CI and contribution tooling | [.github/AGENTS.md](.github/AGENTS.md) |
| Documentation | `docs/README.md` and `docs/package.json`; keep documentation about the implemented product current |

Ordinary frontend work targets **webv2**, served by default; `--webv2` remains a compatibility alias and `--web-legacy` selects legacy web. Verify build/launch targets for UI investigations. The shared `frontend/api` package owns generated OpenAPI/type artifacts.

## Engineering standard

- Inspect implementations, callers, contracts, and tests before designing changes. Define observable outcomes and validation; reproduce bugs where feasible, fix root causes, and reconsider designs that accumulate patches.
- Keep ownership and interfaces clear. Account for edge cases, failures, data lifecycles, and operational limits.
- Every function, dependency, fallback, and test must earn its place. Add abstractions to translate contracts, isolate dependencies, own lifecycles, prevent drift, or simplify callers; avoid pass-through wrappers, speculative extensibility, redundant validation/configuration, and abstractions justified only by similar-looking code.
- Remove obsolete code, tests, and scaffolding; preserve unrelated work. Comment only non-obvious intent, constraints, or trade-offs.
- Test meaningful behavior through its owning interface with independent expectations that catch plausible regressions. Avoid trivial constant/type/re-export assertions, incidental snapshots, duplicate coverage, and mock choreography. Prefer real lightweight dependencies and isolated fixtures; use doubles for expensive/external systems. Optimize test value, not count.
- Keep dependency/lockfile changes intentional. Never weaken assertions, gates, architecture policy, or performance budgets to pass; explain existing failures and verification limits.
- Keep architecture/reference docs aligned with implementation. Skills are optional helpers: adapt them to this stack without importing personal paths or generic ceremony.

## Performance and efficiency

- Inspect affected hot paths and immediate callers for repeated work, renders/subscriptions, waterfalls, queries, serialization/copies, unbounded collections, and retained resources.
- Implement clear, low-risk improvements in scope; larger optimizations must serve the task. Briefly note unrelated opportunities without expanding scope.
- Measure nontrivial optimizations and performance claims with existing budgets and representative fixtures, query counts, profiling, or focused benchmarks. Report material before/after results and limits.
- Preserve correctness and clarity. Require understood invalidation, lifetimes, ordering, and cost for caching, memoization, concurrency, or batching. Do not invent optimizations or benchmarks for a checklist.

## Product quality

Maintain a polished desktop experience: deliberate layouts, responsive interactions, cohesive accessible controls, and complete loading/empty/error/recovery states. Verify changed interactions in the browser; automated checks alone do not establish visual quality.

## Milestones and code review rules

A milestone is one coherent, commit-ready unit. Run relevant checks and self-review every change. Choose review depth by the complete task's behavior, blast radius, and failure consequences—not file/line counts or directory alone. Do not split risky work to avoid review.

| Review depth | When to use it |
| --- | --- |
| **Self-review; no subagents** | Local, low-risk edits with straightforward verification: copy, docs, styling, or an input minimum without shared validation, persistence, or API contract changes. |
| **One independent reviewer** | Substantive features, fixes, or refactors with bounded impact and no high-risk trigger. |
| **Three independent reviewers** | Behavior changes affecting authentication/authorization, account isolation, persisted-data compatibility/migrations, destructive operations, concurrency/resource lifecycles, shared API compatibility, inference numerical/device/memory behavior, or substantial architecture across owners. |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [invoke-ai/InvokeAI](https://github.com/invoke-ai/InvokeAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
