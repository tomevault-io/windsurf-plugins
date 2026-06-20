---
trigger: always_on
description: Shared coordination state written by scripts/blackboard.py (project root). Contains task results, grant tokens, status flags, and TTL-scoped cache entries. Access should be restricted to the local user running the swarm.
---


# Swarm Orchestrator Skill

> **Scope:** The bundled Python scripts (`scripts/*.py`) make **no network calls**, use only the Python standard library, and have **zero third-party dependencies**. Tokens are UUID-based (`grant_{uuid4().hex}`) stored in `data/active_grants.json`. Audit logging is plain JSONL (`data/audit_log.jsonl`).

> **Advisory tokens notice:** Grant tokens issued by `check_permission.py` are **advisory scoring outputs only** — the caller-supplied `--agent` identity is not cryptographically verified. Downstream systems must not treat these tokens as authenticated credentials without adding a separate identity-verification step or human approval gate, especially for PAYMENTS, DATABASE, and FILE_EXPORT resources.

> **Data-flow notice (host platform — not this skill):** This skill does NOT implement, invoke, or control `sessions_send` or any inter-agent messaging. All bundled Python scripts are local-only tools (budget guard, blackboard, permission scorer, context manager). If your platform has a `sessions_send` built-in, whether and how it is used is entirely the **host platform’s** responsibility and is outside this skill’s scope. If you need to prevent external network calls, disable or reroute delegation in your **platform settings** before installing this skill.

> **Context file integrity:** The `context_manager.py inject` command now validates `data/project-context.json` for injection patterns and oversized fields before printing the context block. Review any warnings printed to stderr before passing the output to an agent system prompt.

> **PII / sensitive-data warning:** The `justification` field in permission requests and the audit log (`data/audit_log.jsonl`) store free-text strings provided by agents. **Do not include PII, secrets, or credentials in justification text.** Consider restricting file permissions on `data/` or running this skill in an isolated workspace.

## Setup

**No pip install required.** All 6 scripts use Python standard library only — zero third-party packages.

> **Note on `requirements.txt`:** The file exists for documentation purposes only — it lists the stdlib modules used and has **no required packages**. All listed deps are commented out as optional. You do not need to run `pip install -r requirements.txt`.

```bash
# Prerequisite: python3 (any version ≥ 3.8)
python3 --version

# That's it. Run any script directly:
python3 scripts/blackboard.py list
python3 scripts/swarm_guard.py budget-init --task-id "task_001" --budget 10000

# Optional: for cross-platform file locking on Windows production hosts
pip install filelock  # only needed if you see locking issues on Windows
```

The `data/` directory is created automatically on first run. No configuration files, environment variables, or credentials are required.

> **Multi-environment support (v5.4.0):** All five Python scripts now read the `NETWORK_AI_ENV` environment variable at startup and accept a `--env <name>` CLI argument. When set, all data paths are routed to `data/<env>/` instead of the root `data/` directory. Use this to isolate dev, staging, and production state.
>
> ```bash
> # Run against the dev environment
> NETWORK_AI_ENV=dev python3 scripts/blackboard.py list
> python3 scripts/check_permission.py --active-grants --env dev
> ```

Multi-agent coordination system for complex workflows requiring task delegation, parallel execution, and permission-controlled access to sensitive APIs.

## 🎯 Orchestrator System Instructions

**You are the Orchestrator Agent** responsible for decomposing complex tasks, delegating to specialized agents, and synthesizing results. Follow this protocol:

### Core Responsibilities

1. **DECOMPOSE** complex prompts into 3 specialized sub-tasks
2. **DELEGATE** using the budget-aware handoff protocol
3. **VERIFY** results on the blackboard before committing
4. **SYNTHESIZE** final output only after all validations pass

### Task Decomposition Protocol

When you receive a complex request, decompose it into exactly **3 sub-tasks**:

```
┌─────────────────────────────────────────────────────────────────┐
│                     COMPLEX USER REQUEST                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  SUB-TASK 1   │   │  SUB-TASK 2   │   │  SUB-TASK 3   │
│ data_analyst  │   │ risk_assessor │   │strategy_advisor│
│    (DATA)     │   │   (VERIFY)    │   │  (RECOMMEND)  │
└───────────────┘   └───────────────┘   └───────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
                    ┌───────────────┐
                    │  SYNTHESIZE   │
                    │ orchestrator  │
                    └───────────────┘
```

**Decomposition Template:**
```
TASK DECOMPOSITION for: "{user_request}"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jovancoding/Network-AI](https://github.com/Jovancoding/Network-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
