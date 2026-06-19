---
trigger: always_on
description: >
---


# plan-to-project

Convert a markdown requirements plan into a fully structured, template-compliant
GitHub Project backlog in a single workflow.

## Prerequisites

- `gh` CLI authenticated (`gh auth status`). If not: `gh auth login`
- Python 3.9+ available (`PyJWT` + `cryptography` required for the App-token
  helper; the rest of the skill has no Python deps beyond stdlib)
- Target GitHub org has Issue Types configured: `Project Scope`, `Initiative`, `Epic`,
  `User Story`, `Task` — or pass `--auto-create-issue-types` (FR #46) to have
  preflight create them via GraphQL
- Target GitHub Project V2 has fields: `Priority` (P0/P1/P2), `Size` (XS/S/M/L/XL),
  `Status` (Backlog/In Progress/Done/Blocked)
- Input plan follows KDTIX markdown structure (see [plan-format.md](https://github.com/kdtix-open/skill-plan-to-project/blob/main/references/plan-format.md))
  — and per FR #45, uses the full subsection schema by default

### Auth: personal PAT vs GitHub App installation token (FR #49)

For **Enterprise-owned orgs**, personal fine-grained PATs cannot combine user-scoped
permissions (e.g. "Copilot Requests") with Enterprise-org-scoped permissions
(e.g. `read:project` + `project`). The correct pattern for automation against
Enterprise-owned orgs is a **GitHub App installation token**.

The skill ships a helper:

```bash
# One-time setup
export SDLCA_APP_ID=<App-ID>
export SDLCA_APP_PRIVATE_KEY_PATH=~/.sdlca/<app-slug>.pem
chmod 0600 $SDLCA_APP_PRIVATE_KEY_PATH

# Mint 1-hour installation token (auto-discovers installation for the org)
source scripts/use-app-token.sh kdtix-open
# Exports GH_TOKEN + COPILOT_GITHUB_TOKEN (same value, both aliases set)

# Skill now works against Enterprise-owned org repos
python3 -m scripts.create_issues preflight \
  --org kdtix-open --repo kdtix-open/agent-project-queue --project 7
```

Alternative for non-Enterprise orgs: personal fine-grained PAT or `gh auth login`
with `read:project` + `project` scopes continues to work.

## Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `PLAN_FILE` | Path to markdown plan | `plan-project-plan.md` |
| `ORG` | GitHub org login | `kdtix-open` |
| `REPO` | Target repo (owner/name) | `kdtix-open/my-project` |
| `PROJECT_NUMBER` | GitHub Project V2 number | `8` |

## Installation

### Codex user skill (`~/.codex/skills`)

Install directly with Codex's built-in GitHub skill installer:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo kdtix-open/skill-plan-to-project \
  --path .
```

### Codex native installer CLI (GitHub-backed)

This repo also publishes a native installer entry point so users can install from the
GitHub remote without cloning first:

```bash
uvx --from git+https://github.com/kdtix-open/skill-plan-to-project \
  plan-to-project-install --destination home-skill
```

### Claude Code native installer CLI (GitHub-backed)

Install as a personal Claude Code skill under `~/.claude/skills`:

```bash
uvx --from git+https://github.com/kdtix-open/skill-plan-to-project \
  plan-to-project-install --destination claude-skill
```

### Cursor project rule installer CLI (GitHub-backed)

Install as a repo-local Cursor project rule under `.cursor/rules`:

```bash
uvx --from git+https://github.com/kdtix-open/skill-plan-to-project \
  plan-to-project-install --destination cursor-rule --repo-root /path/to/repo
```

> **Note:** Cursor's official docs support project rules in `.cursor/rules`,
> global user rules in settings, and deeplinks for MCP servers. This repo
> installs the supported project-rule surface because `plan-to-project` is a
> reusable workflow, not an MCP server.

### Codex plugin install

Install as a home-local plugin:

```bash
uvx --from git+https://github.com/kdtix-open/skill-plan-to-project \
  plan-to-project-install --destination home-plugin
```

Install into a chosen repo as a repo-local plugin:

```bash
uvx --from git+https://github.com/kdtix-open/skill-plan-to-project \
  plan-to-project-install --destination repo-plugin --repo-root /path/to/repo
```

> **Note:** The supported repo-local distribution model is a Codex plugin
> (`plugins/` + `.agents/plugins/marketplace.json`). User-scoped skills install under
> `~/.codex/skills`.

## Workflow

### Phase 1 — Pre-flight validation

```bash
python scripts/create_issues.py preflight \
  --org ORG --repo REPO --project PROJECT_NUMBER
```

Validates Issue Type IDs and Project V2 field IDs. Exits with clear error if anything
is missing. Writes `manifest-config.json` with field/type IDs for downstream scripts.

### Phase 2 — Parse plan

```bash
python scripts/create_issues.py parse --plan PLAN_FILE
```

Reads the markdown plan and extracts the 5-level hierarchy (Scope → Initiative →
Epics → Stories → Tasks) with title, description, priority, size, parent reference,
and blocking relationships. Prints a summary for review.

> **For backlogs spanning multiple Epics × multiple Stories per Epic**, see
> [references/sub-agent-fan-out-pattern.md](https://github.com/kdtix-open/skill-plan-to-project/blob/main/references/sub-agent-fan-out-pattern.md)
> for the proven per-layer sub-agent orchestration pattern. Single-agent
> runs against backlogs exceeding 1 Epic × 5 Stories per Epic hit context limits and timeouts.

### Phase 3 — Create issues (top-down)

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kdtix-open/skill-plan-to-project](https://github.com/kdtix-open/skill-plan-to-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
