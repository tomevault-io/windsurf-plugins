---
trigger: always_on
description: This document provides context, patterns, and guidelines for AI coding assistants working in this repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).
---

# AGENTS.md

This document provides context, patterns, and guidelines for AI coding assistants working in this repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## Table of Contents

- [Quick Reference](#quick-reference)
- [Project Structure](#project-structure)
- [Product Overview](#product-overview)
  - [What is ACR](#what-is-acr)
  - [Why This SDK](#why-this-sdk)
  - [Background: BedrockAgentCoreApp](#background-bedrockagentcoreapp)
  - [What agentcore-rl-toolkit Provides](#what-agentcore-rl-toolkit-provides)
  - [Rollout Gateway](#rollout-gateway)
  - [Sandbox SDK](#sandbox-sdk)
  - [Migration Guide (basic_app → rl_app)](#migration-guide-basic_app--rl_app)
  - [Deployment to ACR](#deployment-to-acr)
  - [Evaluation](#evaluation)
- [Environment Variables](#environment-variables)
- [Common Tasks](#common-tasks)
- [Development Tips](#development-tips)
- [External References](#external-references)

---

## Quick Reference

### Key Commands

```bash
# Install dependencies (root package)
uv sync

# Run tests
uv run pytest tests/

# Build and push Docker image to ECR (current approach, may change)
./scripts/build_docker_image_and_push_to_ecr.sh \
  --dockerfile=examples/strands_math_agent/.bedrock_agentcore/strands_math_agent_rl/Dockerfile \
  --tag=latest \
  --context=examples/strands_math_agent

# Run example locally
cd examples/strands_math_agent && uv sync && uv run python rl_app.py
```

### Key Files

| File | Purpose |
|------|---------|
| `src/agentcore_rl_toolkit/app.py` | `AgentCoreRLApp` base class, `@rollout_entrypoint` decorator |
| `src/agentcore_rl_toolkit/client.py` | `RolloutClient` and `RolloutFuture` for training integration and batch evaluation |
| `src/agentcore_rl_toolkit/reward_function.py` | `RewardFunction` base class |
| `src/agentcore_rl_toolkit/rollout_gateway/` | In-repo token-level trajectory capture layer: `RolloutGateway`, `Renderer`, `SamplingBackend`, `TraceRecord` (see [Rollout Gateway](#rollout-gateway)) |
| `src/agentcore_rl_toolkit/sandbox/` | Sandbox SDK: `SandboxClient`, `Sandbox`, `ExecResult` — run shell commands in arbitrary images on ACR (see [Sandbox SDK](#sandbox-sdk)) |
| `sandboxd/` | Go health shim (`agentcore-sandboxd`) that makes arbitrary Docker images satisfy the ACR container contract |
| `examples/strands_math_agent/` | GSM8K math agent example |
| `examples/strands_migration_agent/` | Java migration agent example |
| `examples/strands_officebench_agent/` | OfficeBench office automation agent example |
| `examples/strands_appworld_agent/` | AppWorld API interaction agent example |

---

## Project Structure

```
agentcore-rl-toolkit/
├── src/agentcore_rl_toolkit/
│   ├── __init__.py                 # Public exports
│   ├── app.py                      # AgentCoreRLApp base class
│   ├── client.py                   # RolloutClient for batch evaluation
│   ├── reward_function.py          # RewardFunction base class
│   ├── sandbox/                    # Sandbox SDK (sync client for command execution)
│   │   ├── client.py               # SandboxClient (start/attach) + Sandbox (exec/terminate)
│   │   └── types.py                # ExecResult, SandboxProtocolError
│   └── rollout_gateway/            # Token-level trajectory capture layer (trainer-side)
│       ├── trace.py                # TraceRecord — torch-free output boundary
│       ├── trajectory.py           # TrajectoryManager — per-session message tree
│       ├── render.py               # Renderer protocol; HfTemplateRenderer, TinkerRenderer
│       ├── parsing.py              # tool/reasoning output parsing (sglang optional)
│       ├── gateway.py              # RolloutGateway — assembles the serving unit
│       ├── adapters/               # OpenAI + Anthropic wire protocol adapters
│       └── sampling_backends/      # SamplingBackend impls (vLLM/SGLang HTTP, Tinker SDK)
├── examples/
│   ├── strands_math_agent/         # GSM8K example
│   │   ├── .bedrock_agentcore/     # Dockerfiles for deployment
│   │   ├── basic_app.py            # Production agent
│   │   ├── rl_app.py               # RL-adapted agent
│   │   ├── reward.py               # GSM8KReward implementation
│   │   └── pyproject.toml          # Example-specific dependencies
│   ├── strands_migration_agent/    # Java migration example
│   │   ├── rl_app.py               # RL-adapted migration agent
│   │   ├── evaluate.py             # Batch evaluation script (sync)
│   │   ├── evaluate_async.py       # Batch evaluation script (async)
│   │   ├── reward.py               # MigrationReward implementation
│   │   └── pyproject.toml          # Example-specific dependencies
│   ├── strands_officebench_agent/  # OfficeBench example
│   │   ├── dev_app.py              # RL-adapted office automation agent
│   │   ├── evaluate.py             # Batch evaluation script
│   │   ├── reward.py               # OfficeBenchReward implementation
│   │   ├── tools.py                # Office automation tools
│   │   └── pyproject.toml          # Example-specific dependencies
│   ├── strands_appworld_agent/    # AppWorld example
│   │   ├── rl_app.py               # RL-adapted AppWorld code agent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/agentcore-rl-toolkit](https://github.com/awslabs/agentcore-rl-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
