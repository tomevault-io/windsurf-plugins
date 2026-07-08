---
trigger: always_on
description: n8n community node for **Amazon Bedrock AgentCore harness**. Lets n8n
---

# n8n-nodes-agentcore

n8n community node for **Amazon Bedrock AgentCore harness**. Lets n8n
workflow authors run config-driven AI agents on AgentCore from inside
their workflows without writing infrastructure code.

This file is read by AI coding agents (Claude, Cursor, GitHub Copilot
Workspace, Amazon Q) and by humans onboarding to the repo. Treat it as
authoritative for the conventions below.

## What this package is, and isn't

- It is an n8n community node distributed on npm as `@aws/n8n-nodes-agentcore`.
- It targets the **AgentCore harness** primitive (declarative spec),
  not AgentCore Runtime (the container-hosting primitive).
- It is **not** a general-purpose Bedrock client. It only speaks the
  Harness control plane (`@aws-sdk/client-bedrock-agentcore-control`)
  and data plane (`@aws-sdk/client-bedrock-agentcore`).
- It is **not** a wrapper around the AgentCore CLI. Harness-consumer
  logic is implemented directly on the AWS SDK v3 clients.

## Repo layout

```
.
├── credentials/
│   └── AgentCoreApi.credentials.ts   # n8n credential type: AWS keys + region + execution role ARN
├── nodes/
│   └── AgentCoreHarness/
│       ├── AgentCoreHarness.node.ts          # Node entrypoint: describes UI, implements execute()
│       ├── AgentCoreHarness.node.json        # n8n codex metadata (categories, docs URLs)
│       ├── agentcore.svg                     # Node icon (copied to dist/ by gulp)
│       ├── descriptions/
│       │   ├── Common.ts                     # Shared `toolsField` (fixedCollection for all 6 tool types)
│       │   └── HarnessFields.ts              # Full single-operation field set (model, memory, skills, auth, provisioning)
│       └── helpers/
│           ├── client.ts                     # Credential resolution + VPC config + waitForHarnessReady polling
│           ├── model.ts                      # buildModelConfig() multi-provider union
│           ├── memory.ts                     # buildMemoryConfig()/Update() managed/BYO/disabled
│           ├── environment.ts                # VPC network + filesystem mounts + container artifact
│           ├── skills.ts                     # buildSkillsArray() aws/git/s3/path
│           ├── versioning.ts                 # list versions, upsert named endpoint
│           ├── oauth.ts                       # raw-HTTPS Bearer invoke + event-stream decode
│           ├── stream.ts                     # InvokeHarness streaming consumer (+ tool-use input)
│           └── tools.ts                      # buildToolsArray() + configHash() for drift detection
├── docs/
│   ├── SPEC.md                       # The canonical spec - source of truth for v0.x scope
│   └── iam-trust-policy.json         # Execution-role trust policy template (the only IAM policy shipped; permission policies live in the AWS harness security guide, linked from README)
├── examples/                         # Importable example workflows (01-08)
│   ├── 01-mcp-research-agent.json
│   ├── 02-code-interpreter.json
│   ├── 03-multiturn-support.json
│   ├── 04-multi-provider-switch.json
│   ├── 05-oauth-invoke.json
│   ├── 06-skills-agent.json
│   ├── 07-inline-function-roundtrip.json
│   └── 08-vpc-filesystem.json
├── package.json
└── tsconfig.json
```

## Operations

The node exposes one resource (`AgentCoreHarness`) with two operations:

1. **Run Agent** (`run`) - auto-provisions a Harness on first execution
   keyed by the user-supplied **Agent Name**, reuses it on subsequent runs,
   and calls `UpdateHarness` when the configuration hash drifts. This is
   the primary path.
2. **Invoke Existing Harness** (`invokeExisting`) - BYO ARN. For Harnesses
   created via the AgentCore CLI, console, CloudFormation, or Terraform.

The Run Agent lifecycle lives in `runAgent()` in
`AgentCoreHarness.node.ts`. The cache is workflow static data; AWS is the
source of truth, so static-data misses fall back to `ListHarnesses` before
calling `CreateHarness`.

## Build, lint, type-check

```
npm run build         # tsc + gulp icon copy -> dist/
npm run dev           # tsc --watch
npm run lint          # eslint with eslint-plugin-n8n-nodes-base
npm run typecheck     # tsc --noEmit (strict mode)
npm run format        # prettier --write
npm run format:check  # prettier --check (used in CI)
npm run security:audit  # npm audit --audit-level=high --omit=dev
npm run secrets:check   # secretlint
```

Local testing against a real n8n is done via `npm link` into
`~/.n8n/custom/`. See `README.md` "Local development" for the full flow.

## Runtime dependencies - keep this list short

Today the package has **two** production deps:

- `@aws-sdk/client-bedrock-agentcore`
- `@aws-sdk/client-bedrock-agentcore-control`

Both are Apache-2.0 and AWS-maintained. **Adding a runtime dependency is
a security-review event** - it ships into every n8n install that adopts
this node. Before adding one, confirm:

- License is permissive (Apache-2.0, MIT, BSD, ISC). The
  `dependency-review` workflow enforces this.
- It is actively maintained.
- It does not pull in transitive deps with `exec`, `eval`, or native
  bindings unless absolutely necessary.
- It cannot be replaced by a small inline helper.

`n8n-workflow` is a peer dep, supplied by n8n at runtime. Never bundle it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws/n8n-nodes-agentcore](https://github.com/aws/n8n-nodes-agentcore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
