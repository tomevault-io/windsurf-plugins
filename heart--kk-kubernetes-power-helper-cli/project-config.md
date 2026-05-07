---
trigger: always_on
description: **kk – Kubernetes Power Helper CLI**
---

# Agent Guide for `kk` – Kubernetes Power Helper CLI (Function Wrapper)

## Project Name

**kk – Kubernetes Power Helper CLI**

## Goal

`kk` is a lightweight **Bash function** that wraps `kubectl` and:

- Reduces repetitive typing for common Kubernetes tasks
- Adds smart, pattern-based helpers for pods and deployments
- Keeps behavior close to raw `kubectl` while improving ergonomics
- Keeps namespace state **per shell / per tmux pane** using an environment variable

Unlike `kubectl` plugins or standalone binaries, `kk` is:

- Implemented as a single Bash file (e.g. `~/kk.sh`)
- Loaded with `source ~/kk.sh`
- Exposed to the user as a shell function named `kk`

AI assistants (e.g. Codex) should extend and maintain this tool according to the guidelines below.

---

## Philosophy

1. **Simplicity first**

   - One file, minimal magic.
   - Prefer small helpers over complex frameworks.

2. **Smart automation**

   - Auto-select pods/deployments using patterns and `fzf` when available.
   - Avoid forcing the user to type full resource names.

3. **Avoid abstraction leakage**

   - `kk` is a thin layer over `kubectl`.
   - Do not hide or rewrite `kubectl` semantics.

4. **Safe defaults**

   - Respect the current namespace at all times.
   - Avoid dangerous operations by default.
   - Handle patterns safely (no raw injection into `awk`/`grep`).

5. **Unix-style output**

   - Simple, textual output that plays well with `grep`, `less`, `fzf`, etc.
   - No fancy formatting that breaks piping.

---

## Installation & Runtime Model

### How `kk` is installed

Typical usage:

```bash
# 1) Save kk.sh somewhere, e.g.:
cp kk.sh ~/kk.sh

# 2) Add to ~/.bashrc or ~/.zshrc:
source ~/kk.sh

# 3) Open a new shell and use:
kk ns set my-namespace
kk pods
```

### Function, not executable

- `kk` is declared as:

  ```bash
  kk() {
    # dispatcher calling kk_cmd_* functions
  }
  ```

- It is **not** meant to be executed directly as `./kk.sh`.
- All subcommands live in `kk_cmd_<name>()` helpers and are dispatched from `kk()`.

### Per-shell namespace

- Namespace lives in a shell variable:

  ```bash
  DEFAULT_NS="default"
  : "${KK_NAMESPACE:=$DEFAULT_NS}"
  ```

- Each shell / tmux pane:

  - Has its own `KK_NAMESPACE`.
  - Is independent of other shells.
  - Will default to `"default"` if `KK_NAMESPACE` is unset.

- The helper:

  ```bash
  _kk_current_namespace() {
    printf '%s\n' "${KK_NAMESPACE:-$DEFAULT_NS}"
  }
  ```

  is the **single source of truth** for the active namespace inside `kk`.

- **No files are used** to store the *current* namespace.
- **Exception**: Optional configuration files (e.g. `~/.kk-bindings`) may be used to store persistent mappings (e.g. namespace -> kubeconfig), but the active state remains per-shell.

---

## Current Commands & Behavior

All commands implicitly use the namespace from `_kk_current_namespace()`
(which resolves to `KK_NAMESPACE` or `default`).

### Namespace commands

- `kk ns show`  
  Show the current namespace (from `KK_NAMESPACE` or `default`):

  ```text
  Current namespace: my-namespace
  ```

- `kk ns set <namespace>`  
  Set `KK_NAMESPACE` in this shell only:

  ```text
  Set namespace to: my-namespace
  ```

- `kk ns list`

  - Fetch all namespaces via:

    ```bash
    kubectl get ns -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
    ```

  - If `fzf` exists: interactive picker with header showing the current namespace.
  - Otherwise: print numbered list and prompt for index.
  - The selected namespace is written into `KK_NAMESPACE` (no files touched).

### Namespace Binding (Persistent Kubeconfig)

- `kk bind <namespace> <kubeconfig>`
  - Associates a namespace with a specific kubeconfig file.
  - Stored in `~/.kk-bindings` (or `$KK_HOME/.kk-bindings`).

- `kk unbind <namespace>`
  - Removes the binding.

- `kk bindings`
  - Lists all bindings.

- **Behavior**:
  - When `kk ns set <ns>` is called (or `ns` selected via list), `kk` checks for a binding.
  - If found, `KK_KUBECONFIG` env var is set in the shell.
  - If not found, `KK_KUBECONFIG` is unset (defaulting to standard kubectl behavior).
  - All `kubectl` calls are wrapped to inject `--kubeconfig "$KK_KUBECONFIG"` if set.

### Pods & Services

- `kk pods [name-substring-or-regex]`

  - Runs:

    ```bash
    kubectl get pods -n "$NAMESPACE"
    ```

    where `NAMESPACE` comes from `_kk_current_namespace`.

  - If a pattern is provided, filter by pod name (column 1) using **safe `awk`**:

    ```bash
    awk -v p="$pattern" 'NR==1 || $1 ~ p'
    ```

- `kk svc [name-substring-or-regex]`

  - Runs:

    ```bash
    kubectl get svc -n "$NAMESPACE"
    ```

  - If pattern is provided, filters via the same safe `awk` pattern.

### Pod & Deployment selection (internal helpers)

- `select_pod_by_pattern(pattern)`

  - Lists pod names in the current namespace:

    ```bash
    kubectl get pods -n "$NAMESPACE" \
      -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
    ```

  - Filters with **safe `grep`**:

    ```bash
    grep -E -- "$pattern" || true
    ```

  - Behavior:
    - 0 matches: print clear error and return non-zero.
    - 1 match : print that pod name and return success.
    - > 1 match:
      - If `fzf` exists: interactive selection with `fzf`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heart/kk-Kubernetes-Power-Helper-CLI](https://github.com/heart/kk-Kubernetes-Power-Helper-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
