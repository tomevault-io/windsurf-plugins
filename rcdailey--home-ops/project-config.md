---
trigger: always_on
description: These rules prevent immediate cluster failures. Violations cause crashes, data corruption, or GitOps
---

# Home-Ops Directives

## Tier 1: Breaking Rules

These rules prevent immediate cluster failures. Violations cause crashes, data corruption, or GitOps
drift.

### GitOps Mindset

**Every cluster change MUST flow through git.** Imperative commands are diagnostic only.

- **NEVER run git commit/push without explicit user request** - GitOps requires user commits for
  accountability. This includes using the commit subagent. Always wait for explicit "commit"
  request.
- **NEVER delete resources as a fix** - Deleting jobs, pods, or PVCs treats symptoms, not causes.
  Find the manifest issue and fix it. Exception: cleanup after root cause is fixed.
- **NEVER adjust health probes to fix failures** - Probes detect problems, they don't cause them.
  Investigate WHY the probe fails (resource exhaustion, slow startup, missing deps).

### Troubleshooting Approach

1. **Query**: Gather symptoms via subagent (alerts, logs, events, pod status)
2. **History**: `git log -p --follow --invert-grep --author="renovate" -- path/to/file.yaml` for
   recent changes
3. **Analyze**: Read manifests, check CRD specs, verify dependencies
4. **Research**: Subagent for reference repos, Context7, upstream docs
5. **Fix**: Modify manifests to address root cause
6. **Validate**: `pre-commit run --files <changed-files>`

Recurring issues indicate incomplete root cause analysis.

**All cluster queries MUST use `hops` commands** (`./scripts/hops.py`). Direct use of kubectl,
talosctl, helm, flux, and other cluster CLIs is prohibited except when `hops` lacks the needed
functionality (see escape hatch below). `hops` produces LLM-optimized, token-compact output by
design; raw CLI output wastes context on noise the LLM has to parse and discard.

**`hops` escape hatch:** `hops` is not feature-complete. When a command you need does not exist,
produces too much or too little output, or has a bug: (1) load the `hops` skill, (2) update or add
the command, (3) test the updated command, (4) use it to continue your original task. Do not work
around gaps by falling back to raw CLIs; fix the tool instead. If the gap is too complex to fix
inline, document it as a TODO in the relevant hops source file and fall back to the raw CLI for that
specific operation only.

**`hops` stewardship obligation:** The escape hatch handles missing capability. The no-passthrough
rule (`hops` commands MUST embody investigative workflows, not reformat single CLI calls) requires
active stewardship. While running existing `hops` commands during diagnosis, if any of the following
signals appear, MUST load the `hops` skill and act on them within the current session. MUST NOT
defer the fix to a follow-up session.

- A command's output is barely more useful than the raw kubectl/talosctl/flux call it wraps
- You instinctively reach for a second command to fetch context the first should have included
- A resolver rejects an edge case (orphan pod, terminated workload, fuzzy name) the caller would
  reasonably expect to work
- You catch yourself repeating a sequence of `hops` calls that should be a single command

These signals are fix triggers, not curiosities. Acting on them while unrelated work is fresh in
context is cheaper than a future audit.

### Storage, Volumes, and Resource Patterns

- **RWO volumes MUST use strategy: Recreate** - RollingUpdate causes Multi-Attach errors during pod
  transitions (ceph-block is RWO)
- **RWO volumes REQUIRE advancedMounts** - Single-pod exclusive access requires explicit
  controller/container specification
- **Jobs/CronJobs with RWO PVCs MUST use native sidecar pattern** - initContainers with
  restartPolicy: Always prevents Multi-Attach errors on subsequent runs
- **NEVER specify metadata.namespace in app resources** - Breaks namespace inheritance from parent
  kustomization.yaml
- **App ks.yaml (Flux Kustomization) uses spec.targetNamespace** - Exception to inheritance rule,
  NOT metadata.namespace
- **NEVER use chart.spec.sourceRef for app-template** - Use chartRef (references OCIRepository).
  Exception: External HelmRepository charts may use chart.spec.sourceRef.
- **chartRef REQUIRES namespace for cross-namespace OCIRepository references** - App-template
  OCIRepository is in flux-system namespace; all HelmReleases MUST specify namespace: flux-system

### Secrets and Configuration

- **NEVER use secret.sops.yaml files** - Obsolete pattern replaced by ExternalSecret with Infisical
  ClusterSecretStore
- **NEVER use postBuild.substituteFrom for app secrets** - Timing race condition with ExternalSecret
  creation causes failures
- **ONLY use postBuild.substituteFrom for**: cluster-secrets, email-secrets (pre-existing SOPS
  secrets managed centrally)
- **NEVER use raw ConfigMap resources** - ALWAYS use configMapGenerator in kustomization.yaml with
  files from config/ subdirectory
- **NEVER inline VRL source in vector.yaml** - Separate VRL file required for testing and validation
- **ALWAYS include test data for VRL validation** - Use ./scripts/test-vrl.py for validation

### Scaling

- NFS-dependent apps use native HPAs with external metrics (probe_success from prometheus-adapter)
  to scale 0-1 based on NFS availability

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcdailey/home-ops](https://github.com/rcdailey/home-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
