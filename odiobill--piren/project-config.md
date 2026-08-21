---
trigger: always_on
description: These instructions apply to agents working in this source repository:
---

# Piren Agent Instructions

These instructions apply to agents working in this source repository:

```text
/home/davide/src/piren
```

They are stable project implementation rules. Keep phase-specific handoff details in the vault handoff prompt, not here.

## Required context before coding

Before non-trivial implementation work:

1. Load relevant skills:
   - `test-driven-development` for production behavior changes.
   - `pi-coding-agent-extensions` for Pi extension, CLI, tool, smoke, or package work.
2. Read the Piren vault project docs, at minimum:
   - `/mnt/nas/Piren/Projects/Piren/index.md`
   - `/mnt/nas/Piren/Projects/Piren/knowledge-lifecycle.md`
   - `/mnt/nas/Piren/Projects/Piren/implementation-plan.md`
   - `/mnt/nas/Piren/Projects/Piren/handoff-prompt.md`
3. For architecture or authority-boundary changes, also read:
   - `/mnt/nas/Piren/Projects/Piren/architecture.md`
   - `/mnt/nas/Piren/Projects/Piren/bootstrap-config.md`
   - `/mnt/nas/Piren/Projects/Piren/vault-protocol.md`
   - `/mnt/nas/Piren/Projects/Piren/runtime-placement.md`

## Product thesis to preserve

Piren is not only an agent launcher or task queue. It is a vault-backed team knowledge substrate for a stewarded team of agents, merging LLM-Wiki and Second Brain workflows with explicit multi-agent task execution.

Until the first release candidate, preserve the thesis in:

```text
/mnt/nas/Piren/Projects/Piren/knowledge-lifecycle.md
/mnt/nas/Piren/Projects/Piren/decisions/ADR-0010-vault-as-team-knowledge-substrate.md
```

Every non-trivial task should consider its knowledge delta. Update the minimum useful durable artifact, not everything.

Preferred artifact order:

```text
raw task/session evidence
  -> summary or result
  -> project log
  -> current project docs or handoff
  -> ADR, runbook, wiki page, or skill candidate
```

Raw task/session traces are evidence. Current project docs and ADRs are synthesized truth.

## Architecture boundaries

Keep Piren v1 boring:

- Explicit vault tools, not transparent shell/file interception.
- One file per task.
- Append-only logs where practical.
- External gateway later, through a separate RPC process.
- Pi runtime requirement: local `pi` must be installed on PATH. No npx runtime fallback.
- No hidden memory mutation.

Do not add default automatic inbox polling to interactive `piren run`. Polling belongs only to opt-in worker mode:

```text
piren worker
PIREN_WORKER=1
```

Worker polling must only run for agents explicitly allowed by local installation policy.

## Authority boundaries

Local installation authority lives outside the vault:

```text
~/.config/piren/config.yml
```

This is where these belong:

```yaml
vault_root: /path/to/vault
allowed_agents:
  - piren
excluded_agents:
  - other-agent
```

Agent-local preferences live in:

```text
team/<agent>/config.yml
```

Use agent-local config only for runtime preferences such as model and polling. Do not put `allowed_agents` there.

Piren-owned machine-local secrets or config belong under `~/.config/piren/`. Provider credentials and custom model definitions stay provider-native, for example Pi under:

```text
~/.pi/agent/auth.json
~/.pi/agent/settings.json
~/.pi/agent/models.json
```

Do not put `.env` under `team/<agent>/`.

Do not put `AGENTS.md` under `team/<agent>/`. Piren identity is `SOUL.md`. Project `AGENTS.md` files belong in source repositories like this one.

## Integrated web UI scope

The integrated web UI is intentionally minimal per ADR-0012. It is an emergency interface, not a primary workspace. It provides agent selection, chat, steering, approval gates, a read-only vault browser, and a read-only context indicator. It does NOT provide model or thinking controls: those belong in `team/<agent>/config.yml`, the single source of truth. The model/thinking/agent-switch API routes remain available for external integrations. Rich external solutions (Open WebUI-compatible, purpose-built dashboards) can be built on the HTTP API.

## Extensibility

Piren core is minimal. Additional capabilities come from Pi packages (ADR-0013), declared in `~/.config/piren/config.yml` and loaded as additional `--extension` flags. Vault skills (ADR-0014/ADR-0028) provide reusable procedures stored in `vault/skills/`, future group-scoped `agent-groups/<group>/skills/`, and `team/<agent>/skills/`, injected into agent context at startup.

## Development workflow

Use strict TDD for production behavior changes:

1. Write one failing test for the next tracer bullet.
2. Run the specific test and confirm the expected failure.
3. Implement minimal production code.
4. Run the specific test and confirm it passes.
5. Run the full verification baseline.
6. Update README and relevant vault docs.
7. Search for stale baselines and stale next-step wording.

Keep core logic testable without live Pi auth. Use fake filesystem tests and the fake Pi harness for extension behavior.

Separate core logic from Pi adaptation:

- Core modules should be callable directly from tests.
- Pi extension code should mostly adapt registered tool params to core helpers.
- Avoid hiding important behavior in lifecycle hooks when it can be tested directly.

## Verification commands

From repository root:

```bash
cd /home/davide/src/piren

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Odiobill/piren](https://github.com/Odiobill/piren) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
