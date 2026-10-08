---
trigger: always_on
description: Sandbox escape benchmark for LLM capability evaluation. The project
---

# AgentEscapeBench

Sandbox escape benchmark for LLM capability evaluation. The project
was rebranded from "EscapeBench" to "AgentEscapeBench" — brand-prose
only. The Python package, the Inspect task ID, `escapebench.yaml`,
and infra identifiers (public bucket, Artifact Registry, K8s names,
in-VM paths) all stay as `escapebench`. GitHub repo:
<https://github.com/safety-research/agent-escape-bench>.

## Project setup

- **Package manager**: uv (not pip). Use `uv add <pkg>` to add dependencies, `uv sync` to install.
- **Python**: project supports `>=3.11` (`pyproject.toml`); ruff lints against `py312`. uv picks the lowest matching version it finds installed, so a fresh `uv sync` may pin you to 3.11 — explicitly request 3.12 if you want the lint target.
- **Build system**: hatchling. Source in `src/escapebench/`.
- **Worktree gotcha**: `.env` is gitignored, so a fresh `git worktree add` doesn't carry it over. uv discovers the parent repo's `.env` by walking up from the worktree's pyproject.toml, which means `INSPECT_LOG_DIR=./logs/eval` resolves against the *parent* repo and `.eval` files land outside the worktree. Symlink or copy `.env` into the worktree if you want logs local to it.
- **`.claude/settings.local.json`** is gitignored (per-user agent permissions); shared agent settings go in `.claude/settings.json`.

## Common commands

```bash
uv run pytest              # Run tests
uv run pytest -v           # Run tests verbose
uv sync --extra dev        # Install dev dependencies
uv add <package>           # Add a dependency
uv run ruff check src/     # Lint
uv run ruff format src/    # Format
```

## Code quality

- **Linter/formatter**: ruff (configured in `pyproject.toml` under `[tool.ruff]`)
- **Pre-commit hooks**: ruff lint (`--fix`) + ruff format, auto-run on `git commit`
- After cloning, run `uv sync --extra dev && uv run pre-commit install` to set up hooks
- **Design**: Use proper types and framework idioms. Prefer typed parameters (e.g., `str | list[str]`) over string parsing hacks (e.g., comma-split strings). Leverage framework-native capabilities instead of reimplementing them.

## Dropping a CVE / variant

When a CVE or variant is investigated and **not shipped**, record the
outcome in `docs/dropped-cves.md` before closing the session. Include:
what was tried, the empirical evidence (errnos, kernel/runtime versions,
what mitigation blocked it), what would be needed to revive it, and a
one-line summary at the top. Check this file first before proposing a
new CVE — it's the durable memory across Claude sessions so nobody
re-investigates something we already ruled out.

## Pinned reference exploits

Some reference exploits embed a pre-built binary artifact that's
tied to a specific image variant's kernel vermagic (right now: just
`DOCKER_CAP_SYS_MODULE`, which ships a kernel module as base64 inside
the `.sh`). The binary cannot live inside the qcow2 (the agent-visible
eval VM shares the same image) and cannot be a GCS sidecar that the
model could discover, so it's inlined in the `.sh` and regenerated on
demand by a helper script. The pattern:

- **Author-time artifacts** (`scripts/<name>_helper/`): source + Makefile
  for the binary. Not model-visible.
- **Rebuild script** (`scripts/rebuild_<name>_exploit.sh`): spawns a
  throwaway k8s pod using `escapebench-qemu`, boots the pinned qcow2
  with `-snapshot` (no disk mutation), compiles the binary against the
  VM's running kernel headers, strips debug info, and `sed`-replaces
  the `KO_B64` / `KO_BUILT_FOR_CACHE_KEY` / `KO_BUILT_FOR_KERNEL`
  markers in the `.sh` in place.
- **Staleness check** (`src/escapebench/exploits/registry.py`): register
  the exploit in `_PINNED_REFERENCE_EXPLOITS`. At load time, the loader
  compares the embedded cache-key marker to the live variant cache key
  and raises a `RuntimeError` pointing at the rebuild command if they
  drift — so a forgotten regeneration surfaces as a clear, per-sample
  error the next time the reference is used.

Regeneration workflow when a variant's cache key bumps (kernel / Packer
template / provisioner change):
1. `python -m escapebench.build <variant>` — rebuilds the qcow2.
2. `scripts/rebuild_<name>_exploit.sh` — rebuilds the embedded binary,
   updates the `.sh`.
3. Commit the image-config change and the updated `.sh` together.

## GKE Cluster

Evals run on a GKE cluster with nested virtualization. Each eval sample
gets its own k8s pod running QEMU with KVM acceleration. Topology, k8s
sandbox plugin wrapping, and SSH key handling live in
[`docs/cluster-design.md`](docs/cluster-design.md); WIF / GitHub
Actions service-account setup is in
[`docs/wif-setup.md`](docs/wif-setup.md).

```bash
# Consumer: cluster + KVM pool + node SA with logging/monitoring only.
# Default escapebench.yaml already points at the public pod image and
# the public qcow2 cache, so no further setup is needed to run evals.
./scripts/setup-gke.sh <gcp-project-id>

# Producer: also creates the named GCS bucket (public-read) and grants
# the node SA write access. Required to build your own qcow2 images.
./scripts/setup-gke.sh <gcp-project-id> --bucket gs://<your-bucket>

# Custom cluster name (default: escapebench). The node SA is
# named "<cluster>-node", so distinct names get distinct SAs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [safety-research/agent-escape-bench](https://github.com/safety-research/agent-escape-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
