---
trigger: always_on
description: Repo-wide guidance for GitHub Copilot. Loaded automatically by Copilot
---

# GitHub Copilot Instructions — Sentinel-As-Code

Repo-wide guidance for GitHub Copilot. Loaded automatically by Copilot
(VS Code, GitHub.com cloud agent, code review) on every chat request
in this workspace. Path-scoped instructions live under
[`.github/instructions/`](./instructions/) and stack on top of this file.

For the full Copilot setup map (agents, prompts, instructions),
see [`Docs/Development/GitHub-Copilot.md`](../Docs/Development/GitHub-Copilot.md).

---

## What this repo is

Sentinel-As-Code is a complete CI/CD solution for deploying Microsoft
Sentinel and Defender XDR content from a Git repo. It provisions
infrastructure (Bicep), deploys Content Hub solutions, custom analytical
rules, hunting queries, watchlists, playbooks, workbooks, parsers,
automation rules, summary rules, and Defender XDR custom detections.

Two equivalent CI/CD platforms are supported:

- **GitHub Actions** — under `.github/workflows/`
- **Azure DevOps Pipelines** — under `Pipelines/` (the source of truth
  the GitHub workflows mirror)

Start any unfamiliar task by reading [`Docs/README.md`](../Docs/README.md).

## Repository layout (where things live)

| Folder | Contents | Authoring guide |
| --- | --- | --- |
| `AnalyticalRules/` | Custom Sentinel analytical rules (YAML) | [Docs/Content/Analytical-Rules.md](../Docs/Content/Analytical-Rules.md) |
| `HuntingQueries/` | Custom hunting queries (YAML) | [Docs/Content/Hunting-Queries.md](../Docs/Content/Hunting-Queries.md) |
| `DefenderCustomDetections/` | Defender XDR custom detections (YAML) | [Docs/Content/Defender-Custom-Detections.md](../Docs/Content/Defender-Custom-Detections.md) |
| `Watchlists/` | Reusable data lists (`watchlist.json` + `data.csv` per alias) | [Docs/Content/Watchlists.md](../Docs/Content/Watchlists.md) |
| `Playbooks/` | Logic App playbooks (ARM templates, JSON) | [Docs/Content/Playbooks.md](../Docs/Content/Playbooks.md) |
| `Workbooks/` | Workbook gallery JSON | [Docs/Content/Workbooks.md](../Docs/Content/Workbooks.md) |
| `Parsers/` | KQL parser/function YAMLs | [Docs/Deployment/Scripts.md](../Docs/Deployment/Scripts.md#deploy-customcontentps1) |
| `SummaryRules/` | Summary-rule JSON | [Docs/Content/Summary-Rules.md](../Docs/Content/Summary-Rules.md) |
| `AutomationRules/` | Sentinel automation rules (JSON) | [Docs/Content/Automation-Rules.md](../Docs/Content/Automation-Rules.md) |
| `Bicep/` | Subscription-scoped infra templates | [Docs/Deployment/Bicep.md](../Docs/Deployment/Bicep.md) |
| `Modules/Sentinel.Common/` | Shared deployer + KQL discovery helpers (PowerShell module) | [Docs/Deployment/Scripts.md](../Docs/Deployment/Scripts.md) |
| `Scripts/` | Deploy / drift / dependency / bootstrap PowerShell scripts | [Docs/Deployment/Scripts.md](../Docs/Deployment/Scripts.md) |
| `Tests/` | Pester suites (Wave 3 schema + Wave 4 module unit tests) | [Docs/Development/Pester-Tests.md](../Docs/Development/Pester-Tests.md) |
| `dependencies.json` | Auto-derived content dependency graph | [Docs/Operations/Dependency-Manifest.md](../Docs/Operations/Dependency-Manifest.md) |

## Conventions you must follow

### Language and style

- **Spelling**: en-GB throughout (analyse, behaviour, customise,
  organisation, prioritise, recognise). Existing tooling produces en-US
  output — when editing tool output that uses en-US, leave the existing
  text alone but write any new prose in en-GB.
- **No em-dashes** (`—`) in user-visible prose unless they already
  exist in the file you're editing. Prefer hyphens (`-`) or
  parenthetical phrasing.
- **No emojis** in code, commit messages, or files unless the file
  already uses them. Documentation can use them sparingly when they
  add real meaning (e.g. status icons in tables).

### File headers

Every new PowerShell / Bicep / YAML / JSON file should carry a header
that includes the full repo-relative path and a creation date in
DD/MM/YYYY format. Example:

```powershell
#
# Sentinel-As-Code/Scripts/Foo.ps1
#
# Created by <author> on DD/MM/YYYY.
#
```

For YAML / JSON files where a comment header isn't natural (e.g. data
files), skip the header — but for hand-authored content like analytical
rules, include a brief metadata block at the top of the file.

### Commit messages

Conventional commit format: `type(scope): brief description`.

- **Types**: `feat`, `fix`, `refactor`, `perf`, `style`, `docs`,
  `test`, `chore`, `ci`, `build`, `revert`
- **Scope examples**: `(modules)`, `(scripts)`, `(workflows)`,
  `(deps)`, `(rules)`, `(playbooks)`, `(testing)`, `(deploy)`,
  `(drift)`
- **Body**: explain *why* (business / technical justification), *what*
  changed (file list with reasons), how it was tested. Multi-paragraph
  bodies are normal here; trivial one-liners are not.

**Never include in a commit message**:
- References to Claude, Anthropic, AI assistance, ChatGPT, Copilot, or
  any LLM. Including a `Co-Authored-By` trailer for an AI tool.
- "Generated with..." or similar phrases.
- Emojis (unless the existing log already uses them — it doesn't here).

### Pull requests

- Keep PRs small and atomic. Multiple related commits in one PR is
  fine; multiple unrelated changes is not.
- PR titles follow the same conventional-commit format as commit
  messages.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noodlemctwoodle/Sentinel-As-Code](https://github.com/noodlemctwoodle/Sentinel-As-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
