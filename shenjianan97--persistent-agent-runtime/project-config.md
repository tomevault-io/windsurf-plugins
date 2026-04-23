---
trigger: always_on
description: Cloud-Native Persistent Agent Runtime — durable execution for AI agents.
---

# AGENTS.md — Project Navigation

## Project
Cloud-Native Persistent Agent Runtime — durable execution for AI agents.

**Stack:** Java + Spring Boot (API) · Python (worker) · TypeScript + React 19 + Vite (Console) · PostgreSQL · LangGraph. See [ARCHITECTURE.md](./ARCHITECTURE.md) for full stack and rationale.

## Documentation Map
- `docs/product-specs/` — What the system should do (vision, user stories, core concepts)
- `docs/design-docs/` — How to build it
  - `core-beliefs.md` — Architectural invariants
  - `phase-N/design.md` — Primary design doc per phase
  - `langfuse/`, `agent-capabilities/`, `phase-3-plus/` — Cross-cutting and forward-looking
- `docs/exec-plans/active/` and `docs/exec-plans/completed/` — Implementation plans
- `docs/LOCAL_DEVELOPMENT.md` — Local setup and environment

## Services
- `services/api-service/` — Spring Boot REST API ([README](services/api-service/README.md))
- `services/console/` — React SPA ([README](services/console/README.md))
- `services/worker-service/` — Python worker ([README](services/worker-service/README.md))
- `services/model-discovery/` — Model discovery service

## Common Commands

```bash
make help        # list all targets with descriptions
make init        # first-time setup
make install     # install deps across services
make test        # unit tests (fast, no infra)
make e2e-test    # E2E on isolated infra (DB port 55433)
make test-all    # unit + E2E
make start       # live stack: Console :5173, API :8080
make stop        # stop live stack
make status      # background-target status

# Python (worker) — always use the pinned venv:
services/worker-service/.venv/bin/python ...
```

## New Phase Workflow

1. **Spec** → Add phase sections to `docs/product-specs/` (vision.md, user-stories.md, core-concepts.md)
2. **Design** → Create `docs/design-docs/phase-N/design.md`
3. **Plan** → Create `docs/exec-plans/active/phase-N/` with plan.md, progress.md, agent_tasks/
4. **Execute** → Implement per the task specs
5. **Archive** → Move `active/phase-N/` → `completed/phase-N/`
6. **Update status** → Update [STATUS.md](./STATUS.md)

### Task spec detail level

Task specs in `agent_tasks/` are contracts, not implementation blueprints. They define **what**, not **how**.

**Include:** inputs, outputs, API contracts, schema changes, affected files, dependency graph, constraints, existing code to reference as patterns, acceptance criteria as observable behaviors.

**Exclude:** full source code or paste-ready SQL/Java/Python/TypeScript. The implementing agent reads existing code and writes the implementation itself — over-specified plans produce copy-paste work that misses integration bugs.

### Tracks (chunking large phases)

When a phase exceeds ~40 tasks, split into sequential tracks of ~7-10 tasks each. Tracks are spec subsections, may add `track-N-<name>.md` design docs, and each get their own `exec-plans/` subdirectory. Archive per-track; a phase is complete when all tracks are archived. See `exec-plans/completed/phase-2/` for a worked example.

## Agent Skills (Superpowers)

**Non-negotiable when installed.** At conversation start, invoke `using-superpowers` via the `Skill` tool before any other action — including reading files or asking clarifying questions. Before every task, invoke any relevant skill (debugging, TDD, brainstorming, code review). If there's even a 1% chance a skill applies, invoke it. "This is a simple question" and "let me explore first" are not valid reasons to skip — the skill tells you *how* to explore.

Priority: user instructions > skills > default behavior.

## Boundaries

**Never:**
- Use bare `python3` or `uv run` for worker code — always the pinned venv at `services/worker-service/.venv/` (§Local Validation Notes)
- Point tests at the dev DB (port 55432) — tests use `par-e2e-postgres` on 55433 (§Local Validation Notes)
- Link to PRs in third-party repos from commits or PR descriptions (§External Pull Request References)
- Commit or open a PR with unverified Console UI (§Browser Verification (Console Changes))
- Merge without running the narrowest-scope tests that cover your change (§Testing (Mandatory))

**Ask first:**
- Force-push to `main`/`master`, destructive DB operations, shared CI/infra changes, anything that deletes data

**Always:**
- Invoke relevant superpowers skills (§Agent Skills (Superpowers))
- Use `isolation: "worktree"` when parallel subagents could touch overlapping files (§Parallel Subagent Safety)

## Parallel Subagent Safety

When orchestrating parallel subagents via the Agent tool, **always use `isolation: "worktree"`** if there is any chance two agents modify the same file — even different methods in the same file. Without worktrees, concurrent Edit tool calls on the same file can clobber each other (last writer wins, or `old_string` match fails silently).

- Before launching, check "Affected Component / File paths" for overlap — if any file appears in both scopes, use `isolation: "worktree"` on at least one agent.
- After worktree agents complete, merge their branches into the main working tree.
- Only skip worktrees when agents have truly zero file overlap (e.g., Python worker vs React console).

### Browser verification is the orchestrator's job


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shenjianan97/persistent-agent-runtime](https://github.com/shenjianan97/persistent-agent-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
