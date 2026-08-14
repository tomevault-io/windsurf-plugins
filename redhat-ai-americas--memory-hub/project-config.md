---
trigger: always_on
description: MemoryHub is a Kubernetes-native agent memory component for OpenShift AI. See docs/ARCHITECTURE.md for the full architecture and docs/SYSTEMS.md for subsystem inventory.
---

# MemoryHub

## Project Overview
MemoryHub is a Kubernetes-native agent memory component for OpenShift AI. See docs/ARCHITECTURE.md for the full architecture and docs/SYSTEMS.md for subsystem inventory.

## Branch Strategy

Main is protected. Never push directly to main.

Most work targets **feature branches**, not main. PRs to main happen only when a feature branch is ready to land as a cohesive unit.

**Branch hierarchy:**
- `feat/<topic>` -- long-lived feature branches forked from main
- `feat/<topic>/<subtask>` -- short-lived branches forked from a feature branch for incremental work
- `fix/<description>` -- bugfix branches (target main or a feature branch, depending on scope)

**Workflow:**
1. Create or check out the feature branch (`git checkout -b feat/topic`)
2. For sub-work, branch off the feature branch (`git checkout -b feat/topic/subtask`)
3. PR sub-branches into the parent feature branch, not main
4. When the feature is complete, PR the feature branch into main

**PR targeting rules:**
- Sub-branches PR into their parent feature branch
- Feature branches PR into main only when the work is ready to ship
- Hotfixes (`fix/`) may target main directly when urgency warrants it

This applies to all changes including docs, config, and demo code.

## Issue Management
Use the `/issue-tracker` skill for ALL issue operations. Never create issues manually without using the skill -- it enforces our conventions:
- Every issue references a design document
- Every issue starts in Backlog
- Issues flow: Backlog -> In Progress -> Done

## Development Conventions
- Python with FastAPI for services
- Kubernetes Operator in Python (kopf or operator-sdk)
- Red Hat UBI base images only
- FIPS compliance required
- Use Podman, not Docker
- Use Containerfile, not Dockerfile
- PostgreSQL (OOTB, ships with OpenShift) + pgvector for vector search
- PostgreSQL for graph queries (evolution path to dedicated graph DB)
- MinIO for S3/object storage
- MCP server via fips-agents CLI workflow

## Cluster Contexts

This project deploys to the **mcp-rhoai** cluster context. Other clusters are used for unrelated work. All are configured as named contexts in `~/.kube/config`:

- `mcp-rhoai` — MemoryHub's cluster (n7pd5, sandbox5167)
- `kagenti-rhoai` — Kagenti deployment cluster (gs4bz)
- `memory-hub-fips` — FIPS verification cluster (zks6c, sandbox417)

> **Note:** The FIPS context was previously `fips-rhoai` (l78nk, sandbox1834) which expired 2026-06-09. The main context was previously `workshop-cluster`, renamed to `mcp-rhoai` circa 2026-04-17. Old session notes and retros may reference old names.

**Always pass `--context mcp-rhoai`** on every `oc` / `kubectl` command for this project. Do not rely on the current context, and do not switch contexts with `oc login` or `oc config use-context` — that would break whichever session isn't expecting the switch.

```bash
# Correct — explicit context + explicit namespace
oc get pods --context mcp-rhoai -n memoryhub

# Wrong — relies on implicit current context
oc get pods -n memoryhub
```

This extends the existing `-n` namespace rule: explicit context *and* explicit namespace on every command.

## MCP Server (memory-hub-mcp/)
The MCP server lives in `memory-hub-mcp/` and was scaffolded from the fips-agents MCP template. Follow the workflow in order:

1. `/plan-tools` — Design tools, produces TOOLS_PLAN.md (no code)
2. `/create-tools` — Generate and implement tools via parallel subagents
3. `/exercise-tools` — Test from an agent's perspective, refine ergonomics
4. `/write-system-prompt` — Create SYSTEM_PROMPT.md for consuming agents
5. `/update-docs` — Update README and ARCHITECTURE docs
6. `/deploy-mcp PROJECT=memory-hub-mcp` — Deploy to OpenShift with verification

When working in the MCP server, read `memory-hub-mcp/CLAUDE.md` for import conventions, testing patterns, and architecture details. Key points:
- Always use `src.` prefix for imports
- Call decorated tools directly in tests — FastMCP 3's `@mcp.tool(...)` returns the function itself, no `.fn` access needed
- `fips-agents` is a global CLI (pipx), not in the venv
- Fix file permissions before deployment: `find src -name "*.py" -perm 600 -exec chmod 644 {} \;`

### MCP Tool Creation — MUST use fips-agents workflow
**Never create MCP tools by hand or via sub-agents.** Always use the slash command workflow: `/plan-tools` → `/create-tools` → `/exercise-tools`. Sub-agents cannot run slash commands and will skip the scaffold step, producing tools that lack the template's test structure, permission handling, and registration patterns. This was learned the hard way in Phase 2 — tools created by sub-agents had to be entirely redone. The fips-agents scaffold produces materially better tools with proper test coverage and consistent patterns. When adding tools in future sessions, follow this workflow in the main conversation context, not delegated to sub-agents.

## Onboarding and Process

If this is your first session on this project, read [CONTRIBUTING.md](CONTRIBUTING.md) before writing code. It covers repo layout, dev environment setup, PR expectations, and the same-commit consumer audit rule.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redhat-ai-americas/memory-hub](https://github.com/redhat-ai-americas/memory-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
