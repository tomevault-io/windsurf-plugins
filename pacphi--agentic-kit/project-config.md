---
trigger: always_on
description: > Setup, repair, evidence reporting, and supervised execution for coding-agent integrations
---

# agentic-kit

> Setup, repair, evidence reporting, and supervised execution for coding-agent integrations

## Project Overview

A cross-platform, zero-runtime-dependency Node.js CLI (`ak` / `agentic-kit`) integrating
Ruflo, Agentic QE, Claude Code, Codex, and opt-in OpenCode.

**Tech Stack**: JavaScript ES modules (`.mjs`), Node.js 22+, TypeScript checking via `tsc`
**Architecture**: Domain-Driven Design with bounded contexts

## Quick Start

### Installation

```bash
pnpm install
```

### Build

```bash
npm run build
```

### Test

```bash
npm test
```

### Development

Source runs directly; there is no `dev` script or compilation step.

```bash
node bin/agentic-kit.mjs --help --all
pnpm run typecheck
```

## Agent Coordination

### Swarm Configuration

For complex work, the preferred coordination shape is hierarchical. These are
workflow preferences, not evidence that a swarm is running; obey the current
host/session capability and concurrency limits:

| Setting | Value | Purpose |
|---------|-------|---------|
| Topology | `hierarchical` | Queen-led coordination (anti-drift) |
| Max Agents | 8 | Optimal team size |
| Strategy | `specialized` | Clear role boundaries |
| Consensus | `raft` | Leader-based consistency |

### When to Use Swarms

**Invoke swarm for:**

- Multi-file changes (3+ files)
- New feature implementation
- Cross-module refactoring
- API changes with tests
- Security-related changes
- Performance optimization

**Skip swarm for:**

- Single file edits
- Simple bug fixes (1-2 lines)
- Documentation updates
- Configuration changes

### Available Skills

Use `$skill-name` syntax to invoke:

| Skill | Use Case |
|-------|----------|
| `$swarm-orchestration` | Multi-agent task coordination |
| `$memory-management` | Pattern storage and retrieval |
| `$sparc-methodology` | Structured development workflow |
| `$security-audit` | Security scanning and CVE detection |
| `$performance-analysis` | Profiling and optimization |
| `$github-automation` | CI/CD and PR management |

### Agent Types

| Type | Role | Use Case |
|------|------|----------|
| `researcher` | Requirements analysis | Understanding scope |
| `architect` | System design | Planning structure |
| `coder` | Implementation | Writing code |
| `tester` | Test creation | Quality assurance |
| `reviewer` | Code review | Security and quality |

## Execution Model

- **claude-flow** = LEDGER (coordinates: memory, routing, swarm state)
- **Codex** = EXECUTOR (writes code, runs tests, creates files)

**Critical rule:** DON'T STOP after calling claude-flow commands. Coordination commands return instantly — continue immediately with the next implementation step.

## Ruflo + Codex Automated Workflow

Ruflo is the coordination ledger and policy decision point; Codex workers execute code, tests, and commands. A Ruflo coordination call records work but never replaces implementation.

Use `guidance_brain({ mode: "recommend", task: "..." })` when the task can
benefit from Ruflo-specific capabilities. Its live registry is authoritative
for tool presence; registration alone does not prove configuration,
reachability, health, or authorization. If it is not registered, use compatible
`guidance_recommend`, CLI discovery, and repository instructions.

1. **Recall** — search AgentDB memory and relevant ADRs for patterns and constraints.
2. **Inspect** — read source, runtime, dependency, policy, and health state.
3. **Route** — choose the smallest capable topology, agents, skills, and tools.
4. **Plan** — define acceptance criteria, safety envelope, ownership, and validation.
5. **Execute** — Codex workers implement in isolated scopes; Ruflo records coordination.
6. **Test** — run focused tests, regression tests, and failure-path checks.
7. **Validate** — check types, security, policy, compatibility, and artifact integrity.
8. **Benchmark** — compare a source-bound candidate with a source-bound baseline.
9. **Optimize** — improve measured bottlenecks without weakening the safety envelope.
10. **Receipt** — bind claims, evidence, and decisions to exact source/build inputs.
11. **Handoff** — reconcile concurrent work and disclose unresolved limitations.
12. **Publish** — only an independently authorized release gate may publish immutable artifacts.

### Concurrency and authority invariants

- Never allow two writers in one worktree.
- Read-only research agents may share a checkout; writing agents may not.
- A child may drop capabilities but can never add tools, servers, namespaces, network access, spend, concurrency, or delegation depth.
- Cancel dependent and not-yet-started sibling work when policy denies an action or a required dependency fails.
- MetaHarness may benchmark candidates concurrently, but it cannot promote, serve, or expand its own SafetyEnvelope.
- Only the integration agent changes shared manifests or lockfiles.
- Do not auto-commit, push, merge, release, or delete worktrees unless the user authorized that operation.
- Every consequential action must produce a policy decision receipt; production, destructive, spend, and promotion actions may require human approval.

### Repository harness adapter

When tracked repository instructions define a local collaboration harness:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pacphi/agentic-kit](https://github.com/pacphi/agentic-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
