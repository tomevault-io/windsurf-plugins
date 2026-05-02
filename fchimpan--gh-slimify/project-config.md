---
trigger: always_on
description: GitHub Actions recently introduced the lightweight, cost-efficient `ubuntu-slim` runner (1 vCPU / 5 GB RAM, max 15 min runtime).
---

# 🧭 Rule: `slimfy` — GitHub Actions Slim Migration CLI

## 🎯 Background
GitHub Actions recently introduced the lightweight, cost-efficient `ubuntu-slim` runner (1 vCPU / 5 GB RAM, max 15 min runtime).  
Compared to `ubuntu-latest`, it starts faster and is optimized for short, lightweight CI jobs.  
However, it does not support Docker commands, `services:` containers, and it has a strict 15-minute runtime limit.  
Therefore, it’s essential to identify which existing workflows can be safely migrated to `ubuntu-slim`.

Currently, many teams manually inspect `.github/workflows/*.yml` to make migration decisions — this is tedious and error-prone.  
The goal of this project is to **automate that process using a GitHub CLI extension (`gh extension`) and an AI agent**.

---

## 🎯 Goal
Build a CLI tool named **`slimfy`** that automatically detects and safely migrates eligible `ubuntu-latest` jobs to `ubuntu-slim`.

The tool runs inside any GitHub repository via:
```bash
gh slimfy
```

---

## 🧰 Command Overview

| Command | Description | Behavior |
|----------|--------------|-----------|
| `gh slimfy` | Scan workflows | Analyze `.github/workflows/*.yml` and list candidate jobs and line numbers |
| `gh slimfy fix` | Auto-fix | Replace `runs-on: ubuntu-latest` with `ubuntu-slim` for safe jobs |

---

## 🧩 Migration Criteria

A job is considered **eligible for migration** if **all** of the following conditions are met:

1. The job runs on **`ubuntu-latest`**.  
2. It does **not** use `services:` containers (e.g., PostgreSQL, Redis, MySQL, etc.).
3. The job does **not** use the `container:` syntax.

   Jobs defined with a `container:` section run steps inside a Docker container,
   which requires access to the Docker daemon.  
   Since the `ubuntu-slim` runner runs itself inside a container and does not provide
   `dockerd`, nested container jobs are **not supported**.

   Example (not supported):
   ```yaml
   jobs:
     test:
       runs-on: ubuntu-latest
       container:
         image: node:18
       steps:
         - run: node --version
   ```

   Such jobs must remain on `ubuntu-latest` or another VM-based runner.
4. It does **not** use **Docker commands** such as:
   - `docker build`
   - `docker run`
   - `docker compose`
   - `docker login` / `docker push`
5. It does **not** use any **Docker-based GitHub Actions**, including:
   - Actions that explicitly declare `runs.using: docker`
   - Actions defined with `uses: docker://<image>`
   - **Docker-related marketplace actions** that internally invoke Docker, such as:
     - `docker/build-push-action`
     - `docker/login-action`
     - `docker/setup-buildx-action`
     - `docker/setup-qemu-action`
     - Any custom action from the `docker/` organization or equivalent that requires Docker socket access  
     Example:
     ```yaml
     uses: docker/build-push-action@v6
     with:
       context: .
       push: true
     ```
6. The **latest workflow run duration** is **under 15 minutes**.

If any of the above conditions are violated, the job must **not** be migrated to `ubuntu-slim`.

7. Warn When a Workflow Installs Tools Missing in `ubuntu-slim`.

Introduce a new analyzer feature in **`slimfy`** that detects when a workflow explicitly installs developer tools that are **available on `ubuntu-latest` by default** but **not preinstalled** in `ubuntu-slim`.  
Tools that are missing in `ubuntu-slim` are listed in `internal/workflow/missing_commands.go`.

The goal is to alert users that even though these tools can be installed manually,  
migrating to `ubuntu-slim` will require additional setup time or might fail due to missing dependencies.

---

### 💡 Implementation Hint
In the analyzer:
- Inspect each `uses:` field.
- Flag jobs as *non-migratable* if:
  - `uses:` begins with `docker/`
  - or the referenced action metadata (`action.yml`) contains `runs.using: docker`
  - or `uses: docker://...`
- Maintain a small allowlist for standard JavaScript actions:
  - `actions/checkout`, `actions/setup-go`, `actions/setup-java`, etc.

---

## 🔍 Example CLI Output

### Scan Example
```bash
$ gh slimfy
.github/workflows/lint.yml
  - job "lint" (L8) → ubuntu-slim compatible (last run: 4m)

.github/workflows/test.yml
  - job "unit-test" (L12) → ubuntu-slim compatible (last run: 9m)

Total: 2 jobs can be safely migrated.
```

### Auto-fix Example
```bash
$ gh slimfy fix
🛠 Updating .github/workflows/test.yml:8
    runs-on: ubuntu-latest  →  runs-on: ubuntu-slim
✅ 1 file updated
```

---

## ⚙️ Technical Design

### Detection Logic
- Parse YAML workflows for each job’s `runs-on`, `steps.run`, and `services`.
- Perform lightweight string scanning (`docker`, `services:`).
- Retrieve latest workflow run duration via GitHub REST API (`/repos/{owner}/{repo}/actions/runs`).
- Use AI assistance to classify ambiguous commands (e.g., complex Bash scripts).

### Fix Logic
- Modify YAML safely using AST or YAML node manipulation.
- Commit changes or open a PR (depending on user options).

### AI Agent Role
- Disambiguate borderline cases (e.g., scripts that may indirectly use Docker).
- Explain reasoning in CLI output or PR comments.
- Focus on reducing false positives rather than replacing rules.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fchimpan/gh-slimify](https://github.com/fchimpan/gh-slimify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
