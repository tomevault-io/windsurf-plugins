---
trigger: always_on
description: Read `~/AGENTS.md` first for general coding rules. This file only contains
---

# Bobi Agent Instructions

Read `~/AGENTS.md` first for general coding rules. This file only contains
repo-specific guidance for Bobi.

Bobi is an event-driven AI agent framework.

## Reference Docs

- `README.md`: product overview, installation path, architecture summary, and
  user-facing setup docs.
- `skills/bobi.md`: CLI command reference.
- `skills/create-agent.md`: agent team authoring guidance.
- `skills/checklist-execution.md`: the generic worker protocol for running a
  long job from a committed markdown checklist (read once, next item, verify,
  commit) - no engine, no framework code.
- `skills/slack-setup.md`: Slack integration setup.
- `skills/whatsapp-setup.md`: WhatsApp (Meta Cloud API) integration setup.
- `skills/discord-setup.md`: Discord bot integration setup (Gateway, local server).
- `skills/linear-setup.md`: Linear integration setup.
- `docs/EVENT_SERVER.md`: event-server architecture, topics, and security model.
- `docs/SELF_HOSTED_EVENT_SERVER.md`: running your own webhook ingress - tunnel,
  standalone Node event server, or the durable Cloudflare Worker.
- `docs/ADMIN_PROTOCOL.md`: the supervisor sidecar's wire contract (admin
  topics, the nine commands, heartbeat/lifecycle schemas, compatibility
  promise). Versioned by `SUPERVISOR_VERSION`; additive-only pre-1.0.
- `docs/REFERENCE_IMAGE.md`: the published container image
  (`ghcr.io/moda-labs/bobi`) - what it contains, the `--init` requirement, the
  runtime env contract, the `TEAM_DEPS` bake hook, and how it is published.
- `docs/AGENT_STATE.md`: the agent page's state tri-state (`running` /
  `stopped` / `not_responding`), the manager health probe behind it, and the
  status strip's best-effort telemetry segments.
- `docs/AGENT_OVERVIEW.md`: the agent page's read-only composition view
  (`GET .../overview`) and the `script_cache` savings block in the spend
  payload - how automations are counted and how savings are priced.
- `docs/RUNS_VIEW.md`: the unified runs read model behind the agent page's one
  table (`GET .../runs`) - the status vocabulary, the stalled threshold, and
  the rule that one piece of work produces one row.
- `docs/RUN_DRILLDOWNS.md`: opening a run - the debugging transcript view
  (timestamps + tool calls, distinct from `/messages`) and the Details
  payload for runs that have no transcript.
- `docs/MONITORS.md`: monitor scheduler and the `script_cache` token-saving runner.
- `docs/WORKFLOW_ENGINE.md`: workflow state machine, step types, suspend/resume.
- `docs/RUN_RESUME.md`: resuming a stalled workflow run from the agent page -
  why it spawns a process, and where the single-winner claim lives.
- `docs/TOOL_LIBRARY.md`: unified dependency model - declaring tools/skills/MCP
  deps (pinned `install:` vs guide-only), the catalog, and how they bake + verify.
- `docs/OTEL.md`: agent-authored OTLP telemetry (`bobi agent <name> otel`) -
  operator setup, the resource-attribute table, collector bring-up, and the
  write-only per-instance token requirement.
- `docs/SECURITY.md`: overall security model (trust, credentials, prompt-injection).
- `docs/TICKETING_POLICY.md`: Linear/GitHub ticketing conventions.
- `docs/RELEASE_RUNBOOK.md`: release process and checklist.
- `docs/FRONTEND_QA.md`: local frontend QA guidance for Bobi's vanilla web UIs.
- `docs/design-system/`: the Bobi design system - source of truth for
  anything visual on any Bobi surface (palette, type, icon set, components).
- `DESIGN.md`: source of truth for `bobi setup` UX and its offline constraints;
  its visual tokens were superseded by the design system on 2026-07-31.

## First Principles

- Keep the framework generic. Do not bake Moda-specific workflow assumptions
  into `bobi/`.
- Treat agent teams as the distribution unit for domain behavior: prompts,
  roles, workflows, monitors, tools, and context.
- Runtime behavior should read from the installed package image under
  `$BOBI_HOME/agents/<name>/run/package/`, not directly from source packages.
- Credentials belong in runtime `.env` files or environment variables. Never
  commit secrets.

## Coding Standards

General coding, bug-fix, testing, writing, and commit standards are the
house standards; `~/AGENTS.md` points to them. This file carries only
Bobi-specific deltas on top:

- **Real-Claude e2e as acceptance criteria (judgement call).** Bobi's runtime
  runs through a real Claude brain. For a feature whose correctness depends on
  that brain path (session orchestration, turn handling, tool use, resume,
  event delivery through a live session), the acceptance bar includes an
  end-to-end integration test that drives a REAL Claude session, not only the
  deterministic `stub` brain. Follow the "one mechanism, two brains" pattern:
  parametrize the e2e `[stub]+[claude]`, gate the claude leg on the CLI so it
  runs when available and skips otherwise. This is a judgement call per feature,
  usually the implementor's: a brain-agnostic change (process lifecycle, event
  routing, read-model folds, the admin/control plane) is proven by the stub e2e
  and does not need a claude leg - add one only when the real brain is where the
  risk actually lives.

- **Durable state goes through `bobi/fsutil.py`.** Any file bobi must still be

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moda-labs/bobi-agent](https://github.com/moda-labs/bobi-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
