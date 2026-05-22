---
trigger: always_on
description: This repo uses the **n8n-autopilot plugin** for Claude Code.
---

# n8n Workflow Development — n8n-autopilot Plugin

This repo uses the **n8n-autopilot plugin** for Claude Code.
Workflows are TypeScript (Decorator format). Never write n8n JSON by hand.

> **You are in the INTERNAL repo (`neurawork-git/n8n-autopilot-internal`).** The matching public repo is `neurawork-git/n8n-autopilot` at `~/Documents/Repositories/n8n-autopilot-public/`. When the user asks to release / publish / ship to public, follow [`RELEASE_PROCESS.md`](RELEASE_PROCESS.md) — it is the authoritative manual procedure for syncing internal → public.

> **Reference n8nac version: 2.2.1** (minimum 2.2.0). All setup, workspace, and credential flows below target the v2 manager-backed storage model. Single source of truth: `REFERENCE_N8NAC_VERSION` constant in `scripts/setup-check.sh`. Bump procedure: update the constant, sync README badges + `plugin.json`, add CHANGELOG entry.

## Knowledge skills — read BEFORE inventing CLI surface

| Skill | When to read |
|---|---|
| [`n8nac-cheatsheet`](skills/n8nac-cheatsheet/SKILL.md) | Default first stop. "Which command for X?" — curated table covering 60+ common operations across workspace, env, workflows, executions, credentials, recipes, schemas. |
| [`n8nac-reference`](skills/n8nac-reference/SKILL.md) | Raw `n8nac --help` tree, 74 subcommands. Read when the cheat-sheet does not cover a request. Source of truth for "does this command exist?" — if not in `reference.md`, it does not exist. |
| [`n8n-architect`](https://github.com/EtienneLescot/n8n-as-code) (companion plugin) | Schema-First Research, workflow authoring rules, AI/LangChain patterns, Common Mistakes. Owned by Etienne's `n8n-as-code` plugin. |

**Rule:** before running `npx n8nac <cmd> --help` interactively, `grep` the cheat-sheet, then `grep` the reference file. Only fall back to live `--help` if both lookups fail — and treat that as a sign to regenerate the reference (`bash scripts/dump-n8nac-help.sh > skills/n8nac-reference/reference.md`).

## Cheat-Sheet — User asks X → run Y

**Use these BEFORE inventing flags or grepping `--help`.** If the user's intent matches a row, run the linked skill / command verbatim — do not fish through `--help`. If no row matches, ask the user, do not guess.

| User intent | Exact command |
|---|---|
| "find credential <name>" / "which cred is X" / "list Dropbox creds in this project" | `/n8n-autopilot:find-credential <pattern>` (project-scoped by default; add `--type <credType>` or `--project all`) |
| "list n8n projects" / "which projects exist" / "show projects on instance" | `/n8n-autopilot:find-project` |
| "switch workspace to project X" | `npx n8nac workspace set-project --project-name "<X>"` then `/n8n-autopilot:check-mcps` |
| "show active project / instance binding" | `npx n8nac workspace status --json` |
| "build a workflow that does X" | `/n8n-autopilot:build-workflow "<description>"` |
| "deploy <file>.workflow.ts" | `/n8n-autopilot:deploy <file>.workflow.ts` |
| "fix stale credential IDs in workflows" | `/n8n-autopilot:sync-credentials --fix-workflows` (project-scoped) |
| "regenerate inventory / list of nodes" | `/n8n-autopilot:inventory` |
| "data table CRUD" (create/seed/list/drop tables in n8n) | `/n8n-autopilot:data-tables` |
| "pull schemas" / "update node schemas" | `/n8n-autopilot:pull-schemas` |
| "check setup / MCP / instance health" | `/n8n-autopilot:check-mcps` |
| "find a workflow by name" | `npx n8nac find <query> --json` (use `--remote` for instance-side, default = local+remote) |
| "show executions of workflow <id>" | `npx n8nac execution list --workflow <id>` |
| "inspect a specific execution" | `npx n8nac execution get <executionId> --include-data` |
| "resolve workflow URL for UI" | `npx n8nac workflow present <id> --json` |
| "pull workflow from instance" | `npx n8nac pull <id>` |
| "refresh remote state without overwriting local" | `npx n8nac fetch <id>` |

**Multi-project rule:** every credential / workflow operation runs in the **workspace-pinned project's scope**. Verify the pin via `npx n8nac workspace status --json` before any cred-touching operation. If the answer to "is this the right project?" is uncertain, use `/n8n-autopilot:find-project` first.

## Push-Gate (drift protection — enforced by hook)

The `PreToolUse` hook `scripts/push-gate.sh` blocks two operations by default:

1. **`npx n8nac push <file>`** when `npx n8nac list --search <id>` reports status `CONFLICT`, `MODIFIED_BOTH`, `DIVERGED`, or `REMOTE_ONLY` — i.e. remote has changed since the last local fetch.
2. **`npx n8nac resolve <id> --mode keep-current|keep-local|local-wins`** — always blocked, because it overwrites remote with local in one step.

Bypass (single command, only after explicit user authorization that the remote change should be discarded):

```bash
N8N_AUTOPILOT_ALLOW_LOCAL_WINS=1 <re-run the n8nac command>
```

Default reconciliation path when push is blocked:
1. `npx n8nac pull <id>` — remote wins, sync local
2. Re-edit the local file with your intended change
3. `npx n8nac push <id> --verify` again

The hook auto-runs `npx n8nac fetch <id>` before judging status, so the verdict is always against fresh remote state. Workflows without an `id:` field (new creations) are never blocked.

## Setup

**Brand-new repo? One command:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neurawork-git/n8n-autopilot](https://github.com/neurawork-git/n8n-autopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
