---
trigger: always_on
description: Project context, doc map, and operational guidance for agents working in this repo.
---

# AGENTS.md — Heddle agent guidelines

Project context, doc map, and operational guidance for agents working in this repo.

## Behavioral contract

The twelve behavioral rules below are the authoritative behavioral spec for this worktree. If a future checkout includes a root `CLAUDE.md`, read it as additional guidance, but do not block on it when it is absent.

If something goes wrong, the first triage step is which rule was violated:

| Mistake | Rule | Often shows up as |
|---|---|---|
| Silent wrong assumption | 1 (Think before coding) | Working from a guess about an API/browser behavior without verifying |
| Over-engineered solution | 2 (Simplicity first) | A 6-cell counter grid when one summary line was asked for |
| Touched code outside scope | 3 (Surgical changes) | "While I was in there" cleanup that breaks an adjacent feature |
| Drifted from success criteria | 4 (Goal-driven) | Loop terminated on a partial pass instead of the full criterion |
| Used the model where code would do | 5 (Judgment only) | Asked the model to decide a retry policy that a status code already answers |
| Overran budget silently | 6 (Token budgets) | 90-minute debug session with no summarization |
| Averaged contradictory patterns | 7 (Surface conflicts) | Two error-handling styles mixed in one function |
| Wrote without reading | 8 (Read before write) | Marketing CLI snippet with a flag that doesn't exist in `cli_args/` |
| Test covered shape, not intent | 9 (Tests verify intent) | Hardcoded return value that passes the assertion |
| No checkpoint mid-task | 10 (Checkpoint) | Step 4 of a 6-step refactor broken; steps 5+6 piled on top |
| Forked the convention | 11 (Match conventions) | Introduced `ease` into a codebase that uses `cubic-bezier(...)` |
| Claimed "done" without verifying | 12 (Fail loud) | "Mobile verified" without running Phase 4.5 at 375×812 |

## Confidence

When you `heddle capture`, set `--confidence` to your honest estimate (0.0–1.0) of how likely your change is correct end-to-end given what you tested: ≥0.9 only when build + tests + manual verification all passed cleanly, 0.75–0.89 when most signals passed but coverage is partial, below 0.75 when you're shipping a draft or have unresolved warnings.

## Compatibility

Heddle is still moving quickly. Prefer the current model over preserving legacy behavior.

- Do not add backwards-compatibility shims unless the user explicitly asks for them.
- When a model or API changes, it is acceptable to update callers, tests, and docs to the new behavior instead of keeping legacy support.

## Project Overview

Heddle is an AI-native version control system written in Rust. It combines content-addressed storage, immutable history with stable change identifiers, explicit human and agent attribution, hosted namespace/repository control-plane primitives, and an emerging web product for repository intelligence and operations.

**Key files:**
- `README.md` - Top-level product and capability overview
- `Cargo.toml` - Workspace configuration
- `crates/` - Primary Rust implementation
- `docs/` - Architecture, hosted model, roadmap, and future-state plans
- `specs/quint/` - Formal specifications and model-checking assets
- `web/` - Hosted web product

**Current Status:**
- All core VCS commands implemented
- 600+ tests passing (including formal spec and hosted integration coverage)
- Wire protocol for remote sync complete
- Git Projection import/export/sync implemented
- Packed refs, packfiles, shallow clone, hooks, and crypto signing implemented
- Multi-agent parallel materialized threads implemented (`start --path`, `thread list/show/drop`, `actor spawn/list/done`)
- Hosted namespaces, repositories, grants, and content inspection APIs implemented in foundation form
- Web product in progress: marketing site (shipped), hosted inspection and admin surfaces (foundation), request-access funnel (shipped)
- GitHub App integration (`heddlebot`): PR semantic review summaries, webhook handling, OAuth login (foundation)
- Public review surface at `/review/:owner/:repo/pr/:number` with SSE streaming analysis (foundation)
- Compare/review UX on hosted provenance and hosted builds/workflows remain future-state

## Documentation Truth Rules

When editing docs, specs, or web copy, classify capabilities explicitly:

- **Shipped** - implemented and safe to describe as current behavior
- **Foundation in place** - partially implemented or structurally supported, but not yet a complete user-facing product surface
- **Planned** - clearly intended future-state documented in `docs/` or `web/PRODUCT_SPEC.md`

Do not describe a capability as live if it is only mock-backed in the web app or only planned in docs. Future-state positioning is encouraged when it is grounded in the codebase and roadmap, but it must be labeled accurately.

## Agent skills

### Issue tracker

Issues and PRDs are tracked in GitHub Issues for `HeddleCo/heddle`. See `docs/agents/issue-tracker.md`.

### Triage labels

Use the canonical triage labels, with `question` retained as an additional GitHub issue label for general questions. See `docs/agents/triage-labels.md`.

### Domain docs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HeddleCo/heddle](https://github.com/HeddleCo/heddle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
