---
trigger: always_on
description: Orientation for AI agents (and humans driving them) working on **ebfw**. The goal
---

# AGENTS.md — guide for AI coding agents

Orientation for AI agents (and humans driving them) working on **ebfw**. The goal
of this file is to get you from "fresh clone" to "running the binary and making a
correct change" quickly.

- **User-facing docs:** [`README.md`](README.md) + [`docs/`](docs/) (install,
  configuration, egress policies, comparison).
- **Capabilities & what's deferred:** [`ROADMAP.md`](ROADMAP.md).
- **Deep, painful-to-rediscover gotchas:** `CLAUDE.md` (maintainer working notes —
  **git-ignored**, so it may be absent on a fresh clone; if it's there, read it
  before touching eBPF, attribution, or enforcement code).

## What ebfw is, in one paragraph

A single-binary node agent: a `cgroup_skb/egress` monitor **and** an `SSL_write`
uprobe in one process. It reports outgoing domains (DNS + TLS SNI), HTTP request
paths/headers, and new TCP connections, each attributed to the originating pod
(`namespace/name`), and allows/blocks egress by domain / IP / CIDR / port via a
pure policy engine (off / log / enforce). On Kubernetes it's driven by two CRDs
(`EgressPolicy` + `ClusterEgressPolicy`); off Kubernetes it reads a YAML policy
file. See [`README.md`](README.md) for architecture.

## Try the binary fast (standalone / container-host mode)

ebfw isn't tied to Kubernetes — the same agent watches and enforces egress for
every container (and host process) on a plain Docker/containerd host, driven by a
YAML file instead of CRDs. This is the quickest way to "try out the project."

### Step 0 — what can run where

| Task | macOS | Linux (kernel ≥ 5.8, cgroup v2, root) |
|---|---|---|
| `ebfw policy test` (offline policy eval) | ✅ | ✅ |
| Unit tests (`make test`) | ✅ | ✅ |
| Operator build + envtest (`make test-operator`) | ✅ | ✅ |
| **Run the agent (load eBPF, see live egress)** | ❌ | ✅ |

**eBPF cannot build or run on macOS** (Apple clang has no BPF backend). The agent
must run on Linux. The build itself happens *inside Docker* by default, so you
don't need clang locally — but loading the program needs a Linux kernel and root.

### Path A — evaluate a policy with zero setup (anywhere, no root, no kernel)

```bash
go run . policy test --policy examples/policy.yaml \
  --flow 'dst=203.0.113.5 port=443 domain=api.example.com' \
  --flow 'domain=evil.com port=443'
```

This exercises the pure `internal/policy` engine — the same code that drives
enforcement — without touching eBPF. Great first sanity check on any machine.

### Path B — build a host binary and run it on a Linux box

The binary is compiled in Docker (no local clang needed); you only need a Linux
host to *run* it as root.

```bash
# Build the host binary (any Docker host, incl. building cross via buildx):
docker build --target bin --output type=local,dest=out .   # -> out/ebfw
# (or, on a Linux box with clang + libbpf-dev:  make build  -> bin/ebfw)

# Visibility only — print every container's & host process's egress (defaults, no config):
sudo ./out/ebfw

# With enforcement from a YAML policy file (file is the default policy source):
sudo EBFW_ENFORCE_MODE=log    EBFW_POLICY=examples/policy.yaml ./out/ebfw   # annotate, no drops
sudo EBFW_ENFORCE_MODE=enforce EBFW_POLICY=examples/policy.yaml ./out/ebfw  # actually drop
```

Then generate traffic in another shell (`curl https://example.com/foo/bar`) and
watch it attributed. Add `EBFW_OUTPUT=json` for structured lines.

⚠️ **Enforcement is host-wide** (hooks attach at the root cgroup). A
`defaultAction: Deny` policy cuts off the *whole host* — keep `udp:53` reachable
and prefer a blocklist on a real box so you don't lock yourself out of SSH.

### Path C — run the published agent image as a privileged container

```bash
docker run -d --name ebfw \
  --privileged --network host --pid host \
  -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
  -v "$PWD/examples/policy.yaml:/etc/ebfw/policy.yaml:ro" \
  -e EBFW_ENFORCE_MODE=log -e EBFW_POLICY=/etc/ebfw/policy.yaml \
  ghcr.io/dvrkn/ebfw:latest
```

Full standalone reference (config mounts, capabilities instead of `--privileged`,
attribution caveats off-Kubernetes):
[`docs/install.md` → Run standalone](docs/install.md#run-standalone-container-hosts-no-kubernetes).

### Kubernetes path

If you want the full CRD-driven experience, install the Helm chart on a throwaway
k3d cluster — see [`docs/install.md`](docs/install.md) and the automated loop in
[`test/crd.sh`](test/crd.sh).

> A Linux dev/test box (kernel ≥ 5.8, cgroup v2, Docker + k3d) is the only way to
> actually exercise TCP/TLS/HTTP and the verifier. Maintainer-specific box details
> (if any) live in the git-ignored `CLAUDE.md`, not here.

## Build / test cheat sheet

```bash
make test            # unit tests (no eBPF; runs on macOS/Linux) — run this before any commit
make docker          # build the agent image (BPF + Go compiled inside Docker; any host)
make build           # native agent binary (Linux only; needs clang + libbpf-dev)
make generate        # bpf2go (regenerate *_bpfel.go + *.o) — NOT controller-gen

# Operator subset — pure Go, builds & tests on macOS (never imports internal/egress):
make deepcopy        # controller-gen DeepCopy for api/v1
make manifests       # controller-gen CRDs (-> config/crd/bases) + operator RBAC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvrkn/ebfw](https://github.com/dvrkn/ebfw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
