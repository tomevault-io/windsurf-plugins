---
trigger: always_on
description: > **Target: <1000 tokens.** LLM-optimized. See `flatagent.d.ts`, `flatmachine.d.ts`, `profile.d.ts`, `prompt.d.ts` for schemas.
---

# FlatAgents + FlatMachines Reference

> **Target: <1000 tokens.** LLM-optimized. See `flatagent.d.ts`, `flatmachine.d.ts`, `profile.d.ts`, `prompt.d.ts` for schemas.
>
> **Versioning:** All specs and SDKs use lockstep versioning.

## Concepts

**FlatAgent**: Single LLM call. Model + prompts + output schema. No orchestration.
**FlatMachine**: State machine orchestrating agents. States, transitions, conditions, loops, error handling.

| Need | Use |
|------|-----|
| Single LLM call | FlatAgent |
| Multi-step/branching/retry/errors | FlatMachine |
| Parallel execution | `machine: [a, b, c]` |
| Dynamic parallelism | `foreach` |
| Background tasks | `launch` |

## Model Profiles

```yaml
# profiles.yml — agents reference by name
spec: flatprofile
spec_version: "4.1.0"
data:
  model_profiles:
    fast: { provider: cerebras, name: zai-glm-4.6, temperature: 0.6 }
    smart: { provider: anthropic, name: claude-3-opus-20240229 }
  default: fast        # Fallback
  # override: smart    # Force all
```

Agent model field: `"fast"` | `{ profile: "fast", temperature: 0.9 }` | `{ provider: x, name: y }`
Resolution: default → profile → overrides → override

## OAuth Backends (Codex / Copilot)

- `backend: codex` and `backend: copilot` are **explicit-only** (never auto-detected).
- Backend selection precedence remains: constructor `backend` → resolved `model.backend` → auto-detect (litellm/aisuite only).
- `oauth` settings are read from resolved model config; works identically whether model came from inline agent config or profile.
- Auth file precedence:
  - Codex: `oauth.auth_file` → legacy `codex_auth_file` → legacy `auth.auth_file` → `FLATAGENTS_CODEX_AUTH_FILE` → `~/.pi/agent/auth.json`
  - Copilot: `oauth.auth_file` → `copilot_auth_file` → legacy `auth.auth_file` → `FLATAGENTS_COPILOT_AUTH_FILE` → `~/.agents/flatmachines/auth.json`
- Token handling: pre-request refresh on expiry; if refresh fails, re-read auth store once for cross-process refresh; fallback refresh+retry on `401/403`.
- Retries on `429/500/502/503/504` with exponential backoff (no jitter).

## Agent References

`data.agents` values may be:
- String path to a flatagent config
- Inline flatagent config (`spec: flatagent`)
- Typed adapter ref: `{ type: "flatagent" | "smolagents" | "pi-agent", ref?: "...", config?: {...} }`

## LLM I/O & Formatting (Hard Rules)

- Between LLM stages, default to **plain text/Markdown** handoffs.
- Avoid JSON/Jinja-shaped model-to-model output unless required by strict schema validation or a boundary contract (API/DB/file format).
- Why: JSON/Jinja handoffs increase parse fragility and token overhead.
- Keep `input` / `output_to_context` mappings explicit and shallow.
- Push heavy transforms to boundary actions only (final save/write steps).
- Preserve full source text across stages (avoid excerpt-only chains unless explicitly requested).
- Jinja in config (`input`, `output_to_context`, `transitions`, `foreach`, `wait_for`) must be cross-SDK portable:
  - use property access, comparisons, simple conditionals
  - avoid Python-specific features (`.items()`, `|tojson`, `len()`, `isinstance()`, list comprehensions).
- **Never truncate** LLM inputs/outputs silently.
  - If size is a problem: ask user, enforce prompt limits, reject+retry, or add a repair/compaction stage.
- Each transform must have a one-line justification; otherwise remove it.

## State Fields

| Field | Purpose |
|-------|---------|
| `type` | `initial` (entry) / `final` (exit+output) |
| `agent` | Agent to call |
| `machine` | Machine(s) — string or `[array]` for parallel |
| `foreach` | Array expr for dynamic parallelism (`as`: item var, `key`: result key) |
| `launch` / `launch_input` | Fire-and-forget machine(s) |
| `input` | Map input to agent/machine |
| `output_to_context` | Map `output.*` to `context.*` |
| `execution` | `{ type: retry, backoffs: [2,8,16], jitter: 0.1 }` |
| `on_error` | State name or `{ default: x, ErrorType: y }` |
| `transitions` | `[{ condition: "expr", to: state }, { to: default }]` |
| `mode` | `settled` (all) / `any` (first) for parallel |
| `wait_for` | Channel to wait for external signal (Jinja2 template) |
| `timeout` | Seconds (0=forever) |

## Patterns

**Execution types**: `default` | `retry` (backoffs, jitter) | `parallel` (n_samples) | `mdap_voting` (k_margin, max_candidates)

**Transitions**: `condition: "context.score >= 8"` with `to: state`. Last without condition = default.

**Loops**: Transition `to: same_state`. Machine has `max_steps` safety.

**Errors**: `on_error: state` or per-type. Context gets `last_error`, `last_error_type`.

**Parallel machines**:
```yaml
machine: [review_a, review_b]  # Results keyed by name
mode: settled  # or "any"
```

**Foreach**:
```yaml
foreach: "{{ context.items }}"
as: item
machine: processor
```

**Launch** (fire-and-forget):
```yaml
launch: background_task
launch_input: { data: "{{ context.data }}" }
```

**Wait for signal** (checkpoint, exit, resume on signal):
```yaml
wait_for_approval:
  wait_for: "approval/{{ context.task_id }}"
  timeout: 86400
  output_to_context:
    approved: "{{ output.approved }}"
  transitions:
    - condition: "context.approved"
      to: continue
    - to: rejected
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [memgrafter/flatmachines](https://github.com/memgrafter/flatmachines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
