---
trigger: always_on
description: Context for AI coding assistants (Claude Code, Cursor, Codex, Kiro, Copilot, etc.) editing files in `src/agentic_platform/agent/coding_agent/`. Pair this with the repo-root [`AGENTS.md`](../../../../AGENTS.md) for cross-cutting rules.
---

# AGENTS.md — Coding Agent

Context for AI coding assistants (Claude Code, Cursor, Codex, Kiro, Copilot, etc.) editing files in `src/agentic_platform/agent/coding_agent/`. Pair this with the repo-root [`AGENTS.md`](../../../../AGENTS.md) for cross-cutting rules.

> **The operator-facing setup/deploy guide is in [`README.md`](README.md)** — it has the end-to-end CDK deploy walkthrough, environment-variable reference, secret/key handling, teardown procedure, and troubleshooting. If a human ran into a problem standing the agent up, send them there. This file is for *contributors changing the code*.

## What this agent is

A **FastAPI shim around the `claude` CLI**. The Python code in this folder is *not* the coding intelligence — Claude Code is. The server's only job is:

1. accept a JSON payload over HTTP,
2. clone a target repo (optional),
3. shell out to `claude -p --dangerously-skip-permissions --output-format stream-json …`,
4. log the CLI's events to CloudWatch and capture the final `result` envelope.

The HTTP response is an immediate `202 Accepted` with a `task_id`; the actual `claude` run continues in the background inside the AgentCore microVM. When you change something in this folder, hold that mental model. The "brain" lives in the CLI subprocess; your job is to keep the harness around it small, predictable, and safe. If you find yourself reaching for the Python SDK or a multi-step state machine, push back and ask whether the CLI already does it.

## Where things live

### In this folder (`src/agentic_platform/agent/coding_agent/`)

| File | Purpose | Edit when… |
|------|---------|------------|
| `Dockerfile` | Builds the runtime image. Python 3.12 + Node 24 + Claude Code CLI + git + gh. | The runtime contract changes (Python version, CLI version, system deps). |
| `server.py` | FastAPI app, payload parsing, repo cloning, `claude` subprocess driver. | The invocation contract changes or the CLI flags change. |
| `requirements.txt` | Python deps (fastapi, uvicorn, boto3, pydantic). No SDK — we call the CLI directly. | Adding/upgrading a Python library. |
| `__init__.py` | Empty marker. | Don't. |
| `README.md` | **Operator** docs (deploy, env vars, request shape, troubleshooting). | Behavior visible to callers / operators changes. |
| `AGENTS.md` | This file. **Contributor** docs. | Conventions for code changes change. |

### Outside this folder (deployment + infra)

The deployment stack is **CDK**, not Terraform. Older revisions referenced `infrastructure/stacks/agentcore-runtime/coding_agent.tfvars`; that path no longer exists.

| Path | Purpose | Edit when… |
|------|---------|------------|
| [`/cdk/bin/coding-agent.ts`](../../../../cdk/bin/coding-agent.ts) | CDK app entrypoint. Registers cdk-nag and instantiates `CodingAgentStack`. | Adding env-level branching, registering more stacks. |
| [`/cdk/lib/stacks/coding-agent-stack.ts`](../../../../cdk/lib/stacks/coding-agent-stack.ts) | Stack glue. Composes `AgentCoreRuntime` + `ApiKeyFrontDoor` and exposes the CFN outputs. | Adding new stack-level outputs or props. |
| [`/cdk/lib/constructs/agentcore-runtime.ts`](../../../../cdk/lib/constructs/agentcore-runtime.ts) | ECR repo (imported), GitHub PAT secret, the AgentCore Runtime, log groups, Bedrock IAM. | The runtime needs new env vars / IAM / log destinations. |
| [`/cdk/lib/constructs/api-key-frontdoor.ts`](../../../../cdk/lib/constructs/api-key-frontdoor.ts) | API Gateway, the Lambda proxy that calls `InvokeAgentRuntime`, API key, usage plan. | The HTTP front door changes (new method, different auth, throttle limits). |
| [`/cdk/lambda/invoke/index.ts`](../../../../cdk/lambda/invoke/index.ts) | Lambda handler — bundled by `NodejsFunction`. Pass-through to AgentCore. | The Lambda needs to do more than pass-through (it shouldn't). |
| [`/deploy/build-container.sh`](../../../../deploy/build-container.sh) | Builds & pushes the multi-arch image to ECR (creates the repo if missing). | Build-time changes (tag strategy, scan settings). |

When you add a new env var to `server.py`, add it to **all three** places:

1. `os.environ.get(...)` in `server.py`
2. `environmentVariables: {...}` in [`/cdk/lib/constructs/agentcore-runtime.ts`](../../../../cdk/lib/constructs/agentcore-runtime.ts)
3. The "Environment variables" section of [`README.md`](README.md)

## Contracts that must not drift

These are the load-bearing contracts other systems depend on. Don't change them without coordinating.

1. **HTTP shape.** `POST /invocations` accepts an arbitrary JSON object. `GET /ping` returns `{"status": "healthy"}` with HTTP 200. Both live on port 8080. AgentCore's container contract requires both.
2. **Recognized payload fields.** `task_description`, `repo_url`, `max_budget_usd`, `cwd`. Everything else passes through as Jira-style context inside the prompt. Adding a new recognized field is fine; renaming an existing one is breaking and needs a corresponding change in the Jira integration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/sample-agentic-platform](https://github.com/aws-samples/sample-agentic-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
