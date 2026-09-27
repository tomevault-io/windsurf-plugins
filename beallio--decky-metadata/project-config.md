---
trigger: always_on
description: This document is the project-local operating contract for agents working in this
---

# Decky-Metadata — Agent Operating Contract
## Self-Enforcing Agent Protocol

Protocol Version: 2

This document is the project-local operating contract for agents working in this
directory or its subdirectories. It adapts the universal scripting standards from
`project_template` to this repository's actual stack, and it wires in the
`agent-orchestration` plan → implement → review → finalize engine.

Decky-Metadata is a **Decky Loader plugin** for SteamOS / Steam Deck (Steam Gaming Mode):

- **Frontend:** TypeScript / React in `src/*.ts(x)`, bundled by **rollup** into
  `dist/index.js` (the committed plugin artifact). Tooling is **npm / pnpm**.
- **Backend:** the Decky entry point `main.py` plus the `backend/` package,
  using only the Python standard library and the Decky runtime.

There is **no uv project layout** here, but Python backend tests run through
`uv run --with pytest` in an ephemeral environment (`.protocol: TDD_REQUIRED=true`).
The quality gate covers type checking, build, frontend tests, and backend checks.
It is defined in `scripts/orchestration-hooks/quality-gates`.

---

# 1. Session Initialization

Before implementation work, verify state with `pwd`, `ls`, `git status`, and config
inspection, then output this handshake:

```
AGENT_PROTOCOL_HANDSHAKE

Project Root:
Detected Language(s): TypeScript/React, Python
Execution Mode: Project
Git Repository Present: (Yes/No)
Cache Root: /tmp/Decky-Metadata
Protocol Version: 2
Command Wrapper: ./run.sh

Confirmed Policies:
[ ] Top-down planning
[ ] Cache isolation (/tmp, never inside Dropbox)
[ ] Verified filesystem state
[ ] Verified dependency state (package.json / node_modules)
[ ] Verified run wrapper

STATUS: READY
```

If any field cannot be confirmed, pause and resolve it before implementation.

---

# 2. Project Structure

```
AGENTS.md                         # this contract
.protocol                         # machine-readable policy flags
.envrc / run.sh                   # cache-isolation env + command wrapper
package.json / tsconfig.json      # frontend build config (rollup, tsc)
rollup.config.js                  # bundles src/ -> dist/index.js
src/*.ts(x)                       # TypeScript/React frontend
dist/index.js                     # committed build artifact
main.py                           # Decky backend entry point
backend/                          # storage, matching, providers, and other backend modules
plugin.json                       # Decky plugin manifest
scripts/check_tdd.sh              # pre-commit sanity check
scripts/orchestration            # symlink -> agent-orchestration engine
scripts/orchestration-hooks/      # project quality-gates + finalize-release
orchestration.conf                # orchestration engine config (committed)
docs/specs/                       # durable behavior/interface specs
```

Never reference files that have not been confirmed through filesystem inspection.

---

# 3. Command and Cache Policy

All temp files, tool caches, and installs must live under:

```
/tmp/Decky-Metadata/
```

Run project commands through the wrapper so the redirections apply:

```
./run.sh npm ci
./run.sh npm run build
./run.sh npx tsc --noEmit
./run.sh python3 -m py_compile main.py
```

`run.sh` exports `TMPDIR`, `XDG_CACHE_HOME`, `npm_config_cache`, and
`PYTHONPYCACHEPREFIX` under the cache root. **No generated caches, `node_modules`,
or build temp may be committed inside the repo.** (`dist/index.js` is the one
intentionally committed build output.)

---

# 4. Dependency Policy

- Frontend deps come from `package.json` / `package-lock.json`. Install with
  `npm ci` (or `pnpm install --frozen-lockfile`). Never assume a package exists —
  verify it in `package.json` or `node_modules`.
- The backend (`main.py`) targets the Python standard library and the Decky
  runtime. Do not add third-party Python dependencies without an explicit plan
  entry — Decky plugins ship without a package manager on-device.
- If API behavior is uncertain: read the project source, then `@decky/*` or
  official docs, then web search. Speculative code is forbidden.

---

# 5. Execution Protocol

### Mandatory workflow routing

| Trigger | Required start | Mutation boundary |
| --- | --- | --- |
| Implement/change/refactor | `scripts/decky doctor`, then `scripts/decky verify-change [BASE] --explain` | `--device` deploys; `--allow-launch` permits a configured launch fixture |
| Diagnose Deck/log behavior | `scripts/decky doctor --deck`, `scripts/deck/logs.sh audit --json`, `scripts/decky capture` | read-only unless the user requests changes |
| Inspect SteamUI | `scripts/decky steamui snapshot` / `search PATTERN` | snapshots only below `/tmp/Decky-Metadata` |
| Package/send/check | `scripts/decky status --deck`, then `scripts/decky package-push` | require explicit `--build` / `--push` outside an authorized hook |

Use [docs/runbooks/agent-workflow.md](docs/runbooks/agent-workflow.md) for the
detailed flow. Hook and skill installers default to checks/dry-runs and require
`--install`. Device deployment, launch, package copying, and the `dev` to `main`
promotion retain their explicit flags and human gates.

Lifecycle for a modifying task:

```
ANALYZE → PLAN → IMPLEMENT → VALIDATE → COMMIT → DOCUMENT
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beallio/Decky-Metadata](https://github.com/beallio/Decky-Metadata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
