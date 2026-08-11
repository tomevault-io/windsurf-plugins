---
trigger: always_on
description: `lyca/` contains the Python implementation. Core runtime components live in `lyca/core/`, skill interfaces and examples in `lyca/skills/`, and inspection helpers in `lyca/debug/`. The runnable entry point is `lyca/demo.py`. Architecture and behavior specifications are under `docs/`; consult the matching versioned document before changing a core contract. `data/` holds demo/runtime state, including memory JSON, the append-only Chronicle, blob content, the Skill Registry, and the model-facing comp
---

# Repository Guidelines

## Project Structure & Module Organization

`lyca/` contains the Python implementation. Core runtime components live in `lyca/core/`, skill interfaces and examples in `lyca/skills/`, and inspection helpers in `lyca/debug/`. The runnable entry point is `lyca/demo.py`. Architecture and behavior specifications are under `docs/`; consult the matching versioned document before changing a core contract. `data/` holds demo/runtime state, including memory JSON, the append-only Chronicle, blob content, the Skill Registry, and the model-facing compact Skill Catalog projection. Docker Compose bind-mounts this host directory to `/app/data` by default; override it with `LYCA_DATA_HOST_PATH`. The Agent workspace is a separate `/workspace` Docker volume. Treat generated data as state, not source code.

The separate Admin BFF and OneBot bridge may share `data/chronicle/chronicle.jsonl`; Admin BFF read/write request boundaries explicitly refresh the Chronicle reader, which incrementally absorbs complete external JSONL appends by inode and byte cursor. This synchronizes Chronicle visibility only; Task, Memory, and Agent worker projections remain process-local until distributed reconciliation exists.

## Documentation Synchronization

Every completed code change must update the relevant documentation in the same change. For core runtime or contract changes, update the matching versioned design document and `docs/IMPLEMENTATION_STATUS_V0.1.md`; for channel or integration changes, update the corresponding integration document. Keep `AGENTS.md` and `CLAUDE.md` aligned when commands, architecture boundaries, recovery behavior, or repository workflow changes. Document remaining limitations explicitly instead of describing planned behavior as implemented.

## v2.0 Multi-Agent Workflow

The complete rules are in `docs/MULTI_AGENT_WORK_RULES_V2.0.md`. Sub-agents must use a disjoint write set; plans, analysis, and task breakdowns are not implementation, and completion requires written and verified changes. v0.2 core truth and authority remain controlled by Chronicle, Policy, Effect Broker, and Verification; no sub-agent may bypass them or treat model output as state. Changes to code, contracts, or workflows must update the relevant documentation and keep `AGENTS.md` and `CLAUDE.md` aligned. Dangerous operations require explicit confirmation and minimal scope; credentials must stay out of source, logs, and commits; runtime `data/` is persistent state and must not be casually edited or cleaned; uncertain Effects remain auditable and require receipt/reconciliation/verification, never blind retry or a claim of completion.

Tool Registry lifecycle is persistent by default. Startup may migrate only the exact current-digest built-in Chat delivery dependency from `DEPRECATED` to `ACTIVE`; `REVOKED` and mismatched definitions remain closed, and the migration is Chronicle-audited.

Skill routing is two-stage: the model sees only the stable category/number/purpose entries in `data/skills/skill_catalog.md`; the runtime resolves the submitted catalog ID through `catalog.json`, checks lifecycle/digest/live binding, and reads the full Skill Manifest only after selection.

OneBot proactive text sending is a separate `channel.message.send_to` Tool. It is visible only to owner-authenticated admin Chat, requires a numeric `user_id` or `group_id`, and must use the Effect Broker and bridge receipt; QQ nicknames are never treated as addresses. Bridge send routes accept loopback callers by default or require the shared `ONEBOT_BRIDGE_TOKEN` across hosts. Inbound events are raw-logged first, then `onebot.private_user_ids` and `onebot.group_ids` gate Agent/model access; unauthorized events do not create Chronicle/Task work. Group Agent access additionally requires an exact bot `at` by default (`onebot.require_group_mention`); the bot mention is retained in metadata but removed from canonical command/model text, so `@bot /help` parses as `/help`. The bridge starts and reconnects its configured forward OneBot WS listener with bounded backoff, logs the user/group reason for each drop, and normalizes `at` plus attachment/group-upload metadata. Docker Compose starts the bridge as a separate `onebot-bridge` service; `scripts/start.sh` selects its bridge and authenticated OneBot WS routes for Docker or WSL host networking, with HTTP `:3000` only as fallback.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scmmm/LYCA](https://github.com/scmmm/LYCA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
