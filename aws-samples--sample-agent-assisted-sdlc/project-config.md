---
trigger: always_on
description: Agentic SDLC platform on Amazon Bedrock AgentCore. GitHub issues → autonomous coding → pull requests.
---

# Agent-Assisted SDLC

Agentic SDLC platform on Amazon Bedrock AgentCore. GitHub issues → autonomous coding → pull requests.

## Commands

```bash
npm install                          # Install CDK dependencies
npx cdk synth --quiet                # Validate TypeScript + cdk-nag (run before every commit)
npx cdk deploy --all                 # Deploy all stacks
npx cdk deploy {project}-assistant   # Deploy just the coding assistant stack
npx cdk ls                           # List stacks
npm test                             # Jest snapshot tests
```


## Architecture

6 CDK stacks deployed in order: Infra → SourceControl → ProjectManagement → DeveloperMcp → Gateway → Assistant. Config in `sdlc-config.yaml` (copy from `sdlc-config.template.yaml`). Stack names prefixed with `project:` field from config.

Key paths:
- `lib/nested/*-stack.ts` — stack definitions
- `lib/constructs/` — reusable CDK constructs
- `lib/config.ts` — config schema + `getAssistantDir()` mapping
- `coding-assistants/{claude-code,kiro,codex}/` — container images + plugins
- `project-management/shared/assistants/` — per-assistant invocation strategies
- `gateway/developer-mcp-servers/` — developer tool MCP servers (FastMCP)

## Languages by Directory

| Directory | Language | Package Manager |
|-----------|----------|-----------------|
| `lib/`, `bin/` | TypeScript (strict, NodeNext) | npm |
| `gateway/gateway-iam-proxy/` | Node.js (ESM) | npm |
| `project-management/shared/` | Python 3.12 | pip (requirements.txt) |
| `project-management/github/mcp/` | Python 3.12 + Go (github-mcp-server binary) | uv |
| `source-control/github/mcp/` | Python 3.12 + Go | uv |
| `gateway/developer-mcp-servers/` | Python 3.12 (FastMCP) | uv |
| `coding-assistants/*/runtime/` | Python 3.12 (FastAPI health server) | uv or pip |
| `coding-assistants/claude-code/plugin/hooks/` | Bash | — |
| `test-scripts/` | Bash | — |

## Conventions

- License: Apache-2.0. All Python files have `# Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.` + `# SPDX-License-Identifier: Apache-2.0` header.
- CDK constructs use cdk-nag (AWS Solutions checks). Suppress with `NagSuppressions.addResourceSuppressions()` and a justification string.
- Python formatting: no specific formatter enforced. Follow existing style in each file. `ruff check` must pass.
- TypeScript: strict mode, no implicit any.
- Dockerfiles: pin all dependencies (`pip install pkg==x.y.z`, `npm install -g pkg@x.y.z`, `git clone --branch vX.Y.Z`).
- MCP servers use FastMCP with streamable-http transport on port 8000.
- Health servers use FastAPI on port 8080 (`/ping`, `/health`, `/invocations`).
- Python logging: never call `print()`. Import `get_logger` from `shared/log.py` (or the runtime-local duplicate) and emit structured records. Use `%`-style messages (`logger.info("event %s", value)`); pass structured fields via `extra={...}`. The canonical secret-redaction helper is `redact()` from `shared/log.py`. Process-boundary catch-alls (Lambda `handler`, FastMCP tools) use `logger.exception` and return a structured error; everything else raises a named exception (`WorkspaceSetupError`, `RuntimeCommandError`, `TokenError` — see `shared/errors.py`).
- Runtime images that build with `s3_assets.Asset({path: <runtime-dir>})` carry a duplicate `log.py` in the runtime directory. The canonical file is `project-management/shared/log.py`; each duplicate has a "DUPLICATE OF — keep in sync" banner at the top.

## Security Rules

- Never use `git add -A` in skills or scripts — explicitly stage files to avoid committing orchestrator infrastructure (`.dev-claude/`, `hooks/`, `skills/`, `.claude/`, `.mcp.json`, `settings.json`).
- Never set the `agent:start` label from code — it triggers infinite re-invocation.
- All user-controlled input (issue titles, repo names) must be base64-encoded before passing through shell commands. See `assistants/base.py` for the pattern.
- Validate `owner`/`repo` with `_validate_identifier()` before use in any command.
- Scope guard blocks operations on wrong repo/issue/branch. If `project.json` is missing, all MCP calls are blocked (fail-closed).
- Secrets (AWS keys, private keys, GitHub/OpenAI/Slack tokens) in file writes are blocked by `secret-guard.sh`.

## Anti-Patterns (Do NOT)

- Do not put secrets in code (`.pem` files, API keys). Use Secrets Manager.
- Do not use `shell=True` in Python subprocess calls.
- Do not hardcode AWS account IDs or region — read from config or environment.
- Do not add `cdk.out*/`, `cdk.context.json`, `.threatmodel/`, or `security-scans/` to git.
- Do not use `curl ... | bash` without downloading to a file first (supply chain risk).
- Do not expose tokens in process arguments — use credential helpers or temp files.
- Do not deploy with `authorizerType: NONE` on the gateway in production.
- Do not make the OIDC trust policy `repo:*` — always set `allowedRepos` in config.

## Adding a Developer MCP Server (simplest extension)

1. Create `gateway/developer-mcp-servers/<name>/` with `main.py`, `Dockerfile`, `pyproject.toml`, `uv.lock`
2. Add to `sdlc-config.yaml` under `gateway.developerMcpServers`
3. `npx cdk deploy {project}-developer-mcp`

## Adding a Coding Assistant


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/sample-agent-assisted-sdlc](https://github.com/aws-samples/sample-agent-assisted-sdlc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
