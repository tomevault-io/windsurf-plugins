---
trigger: always_on
description: **Rossoctl** is a cloud-native middleware platform for deploying and orchestrating AI agents. It provides framework-neutral infrastructure for running agents (LangGraph, CrewAI, AG2, etc.) with authentication, authorization, trusted identity, and scaling.
---

# CLAUDE.md - Rossoctl Repository

## Project Overview

**Rossoctl** is a cloud-native middleware platform for deploying and orchestrating AI agents. It provides framework-neutral infrastructure for running agents (LangGraph, CrewAI, AG2, etc.) with authentication, authorization, trusted identity, and scaling.

## Quick Start

```bash
# Deploy to Kind cluster
./.github/scripts/local-setup/kind-full-test.sh --skip-cluster-destroy

# Show service URLs
./.github/scripts/local-setup/show-services.sh

# Access UI at http://rossoctl-ui.localtest.me:8080 (see show-services.sh for credentials)
```

## Repository Structure

```
rossoctl/
├── rossoctl/
│   ├── ui-v2/              # React frontend
│   ├── backend/            # FastAPI backend
│   ├── tests/e2e/          # E2E tests
│   └── examples/           # Example agents/tools
├── charts/                 # Helm charts
│   ├── rossoctl/            # Main platform chart
│   └── rossoctl-deps/       # Dependencies
├── deployments/
│   └── envs/               # Environment values
├── .claude/skills/         # Claude Code skills
└── docs/                   # Documentation
```

## Key Commands

| Task | Command |
|------|---------|
| Deploy to Kind | `./.github/scripts/local-setup/kind-full-test.sh --skip-cluster-destroy` |
| Deploy to OpenShift | `scripts/ocp/setup-rossoctl.sh` |
| Run E2E tests | `uv run pytest rossoctl/tests/e2e/ -v` |
| Run linter | `make lint` |
| Pre-commit | `pre-commit run --all-files` |

## Claude Code Skills

Skills in `.claude/skills/` provide guided workflows:

| Category | Skills (invoke with `Skill` tool) |
|----------|--------|
| Kubernetes | `k8s:health`, `k8s:pods`, `k8s:logs` |
| Clusters | `kind:cluster`, `hypershift:cluster` |
| Auth | `auth:keycloak-confidential-client`, `auth:otel-oauth2-exporter` |
| Istio | `istio:ambient-waypoint` |
| OpenShift | `openshift:debug`, `openshift:routes`, `openshift:trusted-ca-bundle` |
| Testing | `tdd:hypershift`, `testing:kubectl-debugging`, `k8s:live-debugging` |
| Git | `git:worktree` |

See [docs/skills/](docs/skills/README.md) for skill index and [docs/ai-ops/](docs/ai-ops/README.md) for workflows.

## HyperShift Cluster Access

HyperShift hosted cluster kubeconfigs are stored at:

```
~/clusters/hcp/<MANAGED_BY_TAG>-<cluster-suffix>/auth/kubeconfig
```

Examples:
- `~/clusters/hcp/rossoctl-hypershift-custom-uitst/auth/kubeconfig`
- `~/clusters/hcp/rossoctl-hypershift-custom-mlflow/auth/kubeconfig`

Use with kubectl/oc commands (auto-approved in settings.json):

```bash
export KUBECONFIG=~/clusters/hcp/rossoctl-hypershift-custom-uitst/auth/kubeconfig
kubectl get pods -n rossoctl-system
```

The management cluster kubeconfig is separate (in `~/.kube/`).

## Worktree Workflow

Run worktree code from main repo (keeps credentials in one place):

```bash
# Stay in main repo
# For HyperShift: source .env.<MANAGED_BY_TAG> (see .github/scripts/local-setup/README.md)
source .env.rossoctl-hypershift-custom

# Run worktree's test script
.worktrees/my-feature/.github/scripts/local-setup/kind-full-test.sh --skip-cluster-destroy
```

## Key Technologies

| Component | Purpose |
|-----------|---------|
| Istio Ambient | Service mesh (mTLS) |
| Keycloak | OAuth/OIDC |
| SPIRE | Workload identity (SPIFFE) |
| Shipwright | Container builds |
| Phoenix | LLM observability |

## Namespaces

- `rossoctl-system` - Platform components
- `keycloak` - Identity provider
- `team1`, `team2` - Agent namespaces

## Protocols

- **A2A**: Agent-to-Agent (Google) - `/.well-known/agent-card.json`
- **MCP**: Model Context Protocol (Anthropic) - Tool integration

## Context Budget (MANDATORY)

**Context window pollution is the #1 cost driver.** Build output, kubectl responses, and
test results dumped into conversation history get re-read on every subsequent turn,
causing exponential cost growth in long sessions. Follow these rules to minimize context usage.

### Rule 1: Redirect command output to files

Any command that produces more than ~5 lines MUST redirect to a session-scoped log file:

```bash
# Set a session-scoped log directory (use worktree/cluster name to avoid collisions)
export LOG_DIR=/tmp/rossoctl/tdd/$WORKTREE   # TDD sessions
export LOG_DIR=/tmp/rossoctl/rca/$WORKTREE   # RCA sessions
export LOG_DIR=/tmp/rossoctl/k8s/$CLUSTER    # K8s debugging
mkdir -p $LOG_DIR

# Pattern: redirect output, return only exit code
command > $LOG_DIR/descriptive-name.log 2>&1; echo "EXIT:$?"
# or
command > $LOG_DIR/name.log 2>&1 && echo "OK" || echo "FAIL (see $LOG_DIR/name.log)"
```

### Rule 2: Analyze logs in subagents

**NEVER read large log files in the main context.** Use subagents:

```
Task(subagent_type='Explore'):
  "Use Grep with context (-C 3) on $LOG_DIR/test-run.log to find FAILED|ERROR.
   Do NOT read the whole file. Return: first error, test name, and 2-3 lines of context."
```

Use subagents for BOTH failure analysis AND success verification (e.g., "verify traces appear in the log").

### Rule 3: Small output is OK inline

These are fine without redirection (produce <5 lines):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rossoctl/rossoctl](https://github.com/rossoctl/rossoctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
