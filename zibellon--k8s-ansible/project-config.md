---
trigger: always_on
description: Production bare-metal Kubernetes cluster automation. Two halves:
---

# k8s-ansible

Production bare-metal Kubernetes cluster automation. Two halves:

- **`playbook-system/`** — imperative, node-scoped bootstrap and lifecycle operations (kubeadm, HAProxy LB as systemd, ETCD encryption, node join/drain/remove).
- **`playbook-app/`** — declarative cluster-scoped application installs via local Helm charts, every component deployed through a consistent 3-phase pattern (`pre` → `install` → `post`).

This file is the **map** of the project; detailed **catalogs** live under `.claude/rules/`. Read section 1 before editing anything.

---

## 0. Hard Invariants (do not violate)

Violating any of these will break the cluster or leak secrets.

- `argocd` and `longhorn-system` namespaces are **upstream-fixed** — never rename.
- `kube-proxy` is **disabled at `kubeadm init`** (`--skip-phases=addon/kube-proxy`). Cilium replaces it. Do not re-enable.
- `hosts-vars-override/` is **never committed**. It contains `ansible_password`, real IPs, Vault unseal keys, and all secrets.
- Always run Ansible with **both** inventories: `-i hosts-vars/ -i hosts-vars-override/`. Running with only one is always a bug.
- **System playbooks require `--limit`**. Forgetting `--limit` on `node-install.yaml` / `cluster-init.yaml` / `manager-join.yaml` / `worker-join.yaml` will fail a `tasks-require-limit.yaml` gate (by design).
- Exactly **one** manager in inventory must have `is_master: true`. That host becomes `master_manager_fact` — the single delegation target for every cluster-scope operation.
- Before adding a new node to the cluster, run `playbook-app/cilium-install.yaml --tags post` first — it refreshes the Cilium host firewall with the new node's IPs, otherwise the join handshake is blocked.

---

## 1. Mental Model in 50 Lines

After this section you should be able to classify any file in the repo and know which concept it belongs to.

### 1.1 Two orthogonal decompositions

The repo is structured along **two axes** that intersect at every component:

1. **Vertical layers** (bottom-up, what you build on top of what):
    ```
    L7 Observability        (Prometheus, Grafana, alerting, ServiceMonitors)
    L6 Applications         (ArgoCD, GitLab, Teleport, Zitadel, …)
    L5 Platform services    (Vault + ESO, cert-manager)
    L4 Storage              (Longhorn)
    L3 Ingress              (Traefik, HAProxy ingress)
    L2 CNI                  (Cilium — replaces kube-proxy, host firewall)
    L1 Control plane        (kubeadm, ETCD+encryption, HAProxy apiserver LB on systemd)
    L0 OS / node            (containerd, runc, kubelet, kernel modules, systemd)
    ```
2. **Horizontal phases** (per-component, ordering inside one install):
    ```
    pre   → install → post
    NP+ESO  workload   ingress/SM
    <c>-pre <c>       <c>-post
    ```
    Each phase is a **separate Helm release** so a single phase can be re-run in isolation.

### 1.2 Two "repos in one"

- `playbook-system/` is **imperative and node-scoped**. Plays target specific hosts (`--limit <host>` is required by a gate). They install packages, write `/etc/…` files, start systemd units. Facts like `is_cluster_init` / `is_node_joined` come from `tasks-gather-cluster-facts.yaml`.
- `playbook-app/` is **declarative-ish and cluster-scoped**. Plays always `hosts: managers` + `gather_facts: false`; all `kubectl`/`helm` work delegates to one manager (`master_manager_fact`) with `run_once: true`.

### 1.3 Two inventory layers

- `hosts-vars/` — base defaults, committed to git. Templates, public values, schema.
- `hosts-vars-override/` — real inventory, secrets, cluster-specific overrides. **Never committed**.

Merge order: `hosts-vars/` → `hosts-vars-override/` → inline play vars. Arrays with the `*_extra` suffix are **concatenated** across layers, not replaced (see §7.2).

### 1.4 One delegation point

`master_manager_fact` is the hostname of the inventory host with `is_master: true`. Every `kubectl` / `helm` invocation in `playbook-app/` is `delegate_to: "{{ master_manager_fact }}" + run_once: true`. This means:

- Only that host needs a kubeconfig (`/root/.kube/config` is created by `tasks-kubectl-configure.yaml` at cluster-init).
- Charts are rsync-copied to that host's `{{ remote_charts_dir }}` (default `/opt/helm-charts/`), then Helm runs against the local filesystem.
- If the master manager goes down, `playbook-app/` operations stop working until you elect another `is_master: true`.

### 1.5 The "where am I?" stack

```
┌─────────────────────────────────────────────────────────────────┐
│  L7 Observability (mon, grafana, kube-state-metrics, node-exp)  │  playbook-app/mon-*
│  L6 Applications (argocd, gitlab, teleport, zitadel, medik8s)   │  playbook-app/
│  L5 Platform    (vault, external-secrets, cert-manager)         │  playbook-app/
│  L4 Storage     (longhorn)                                      │  playbook-app/
│  L3 Ingress     (traefik @ traefik-lb, haproxy @ haproxy-lb)    │  playbook-app/
│  L2 CNI         (cilium — replaces kube-proxy, host firewall)   │  playbook-app/
│  L1 Control pl. (kubeadm, ETCD+encryption, HAProxy apiserver LB)│  playbook-system/
│  L0 OS / node   (containerd, runc, kubelet, modules, systemd)   │  playbook-system/
└─────────────────────────────────────────────────────────────────┘
```

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zibellon/k8s-ansible](https://github.com/zibellon/k8s-ansible) — distributed by [TomeVault](https://tomevault.io/claim/zibellon).
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
