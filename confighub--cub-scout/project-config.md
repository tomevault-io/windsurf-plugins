---
trigger: always_on
description: Read-only Kubernetes observer. Detects ownership (Flux, ArgoCD, Helm, Crossplane, ConfigHub, Native).
---

# cub-scout

Read-only Kubernetes observer. Detects ownership (Flux, ArgoCD, Helm, Crossplane, ConfigHub, Native).

## Read First

For a fresh coding session in this repo, read these in order:

1. [AI-README-FIRST.md](AI-README-FIRST.md)
2. [HANDOVER.md](HANDOVER.md)
3. [docs/reference/commands.md](docs/reference/commands.md)
4. [docs/reference/cli-contract.md](docs/reference/cli-contract.md)
5. [docs/reference/json-contracts.md](docs/reference/json-contracts.md)

## Related Abilities

- `cub-scout`: read-only cluster/GitOps observation, connected comparison, import preview, MCP serving
- `cub`: ConfigHub CLI for intended-state workflows, spaces/units/targets, `cub gitops discover/import`
- `confighub/sdk`: renderer/bridge implementation detail used by `cub`

Important boundary:
- `cub-scout import --git-path` is a local structure/import-preview flow
- `cub gitops import` is cluster discovery + render-target based
- do not claim that `cub-scout` can do SDK renderer work unless current code/help exposes it

## Build & Run

```bash
go build ./cmd/cub-scout
./cub-scout map              # Interactive TUI
./cub-scout map list         # Plain text
./cub-scout trace deploy/x -n y
./cub-scout scan
./cub-scout gitops status    # GitOps pipeline health (v0.14)
./cub-scout graph export     # Resource graph (v0.6)
./cub-scout patterns detect  # Pattern detection (v0.7)
```

**Always use `./cub-scout`** (local binary), not `cub-scout`.

**Output formats (v0.14):** Most commands support `--format ascii|json|md`.

## Claude Capability-Assistant Mode

For demo flow "Can I do X with cub-scout or ConfigHub?":

- Start from [AI-README-FIRST.md](AI-README-FIRST.md) for current tool boundaries and issue queue.
- Start from shared skill profile: [skills/cub-scout/SKILL.md](skills/cub-scout/SKILL.md)
- Use [docs/howto/claude-capability-assistant.md](docs/howto/claude-capability-assistant.md) as the operating playbook.
- If a request is not currently supported, file using:
  - `./scripts/create-ai-capability-issue.sh ...`
  - or issue template: `AI capability gap`

## Documentation

| File | Purpose |
|------|---------|
| [README.md](README.md) | Project overview, install, quick start |
| [CLI-GUIDE.md](CLI-GUIDE.md) | Workflow-first CLI guide |
| [docs/reference/cli-reference.md](docs/reference/cli-reference.md) | Complete command catalog (A-Z) |
| [docs/reference/commands.md](docs/reference/commands.md) | Detailed command usage and examples |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to contribute |
| [docs/semantic-contract.md](docs/semantic-contract.md) | ASCII vs JSON meaning contract (R1-R6) |

## Key Principles

1. **Single cluster** — standalone mode inspects one kubectl context; multi-cluster only via connected mode
2. **Read-only by default** — never modifies cluster state
3. **Deterministic** — same input = same output, no AI/ML
4. **Parse, don't guess** — ownership from actual labels, not heuristics
5. **Complement GitOps** — works alongside Flux, Argo, Helm
6. **Graceful degradation** — works without cluster, ConfigHub, or internet
7. **Test everything** — `go test ./...` must pass
8. **CLI/TUI parity** — CLI and TUI are two renderings of one model. Every feature must have both a CLI command (with `--format ascii|json|md`) and a TUI equivalent. CLI is not a second-class citizen.

## Current Milestone Reality

Use [HANDOVER.md](HANDOVER.md) as the latest execution snapshot.

As of the current handover:
- the Argo truth-and-guidance track is closed (`#365`, `#366`, `#367`)
- the Git import parser track is complete through ApplicationSet generator support (`#363`)
- `#369` is shipped: `doctor` is now the first standalone MCP troubleshooting tool
- `#364` is investigated, not a mandate to merge `cub-scout` with `cub gitops import`
- the highest-leverage open queue is now `#370`, then `#368`

## Directory Structure

| Path | Purpose |
|------|---------|
| `cmd/cub-scout/` | CLI commands, TUI |
| `pkg/agent/` | K8s watcher, ownership detection |
| `test/` | Tests |

## Ownership Detection

| Owner | Detection |
|-------|-----------|
| Flux | `kustomize.toolkit.fluxcd.io/*` or `helm.toolkit.fluxcd.io/*` labels |
| ArgoCD | `argocd.argoproj.io/instance` label or tracking-id annotation |
| Helm | `app.kubernetes.io/managed-by: Helm` |
| Terraform | `app.terraform.io/run-id` annotation or managed label |
| Crossplane | `crossplane.io/claim-name` label *(experimental)* |
| ConfigHub | `confighub.com/UnitSlug` label |
| Native | None of the above |

## Testing

```bash
go build ./cmd/cub-scout
go test ./...
```

## Backlog Tracking

Future ideas live in supplemental planning docs (`docs/roadmap-*.md`, `docs/reference/*-product-guide.md`).
These are non-authoritative and easy to forget during sprint planning.

Rules:
- `docs/roadmap.md` contains an **Untracked Backlog Checklist** at the top
- When a new idea appears in any planning doc, add it to the checklist
- When an item graduates to real work, file a GitHub issue and remove the checklist entry
- The master tracking issue is **#154** — keep it current

This prevents ideas from being trapped in prose nobody re-reads.

---

## Pre-Coding Test & Success Proof Requirements


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [confighub/cub-scout](https://github.com/confighub/cub-scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
