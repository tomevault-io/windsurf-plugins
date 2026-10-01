---
trigger: always_on
description: An **agent** is what the platform spawns one `opencode` subprocess to run.
---

# Agent reference

An **agent** is what the platform spawns one `opencode` subprocess to run.
It is the unit a workflow node references. The database is the source of
truth — there is no `.md` file on disk; the editor in the UI reads/writes
DB columns and serializes the frontmatter into `OPENCODE_CONFIG_CONTENT`
when launching.

## Frontmatter fields

```yaml
# Required
name: code-auditor                  # ^[a-z0-9][a-z0-9_-]*$, max 128
description: Audits a code diff…    # any free-form string
outputs:                            # output port names the agent will emit
  - findings
  - summary
readonly: true                      # see "Concurrency" below

# Optional — overridable per workflow node
model: anthropic/claude-sonnet-4-6  # falls back to config.defaultModel
variant: thinking-2025-09           # opencode model variant
temperature: 0.2                    # [0, 2]
steps: 50                           # opencode step budget
maxSteps: 100

# Optional — attached skills (each loaded into per-run OPENCODE_CONFIG_DIR)
skills:
  - lint
  - typing

# Optional — opencode permission map, passed through verbatim
permission:
  read: '**/*'
  write: '**/*'

# Any other key lands in frontmatterExtra and round-trips on edit
my-extra-key: arbitrary value
```

| Field            | Type             | Default | Notes                                              |
| ---------------- | ---------------- | ------- | -------------------------------------------------- |
| `name`           | string           | —       | URL-safe slug, unique per daemon                   |
| `description`    | string           | `''`    | Shown in lists; not sent to the model              |
| `outputs`        | `string[]`       | `[]`    | Port names the framework expects in the envelope   |
| `readonly`       | `boolean`        | `false` | `true` allows the node to run in parallel with other readonly nodes; `false` takes the per-task write semaphore |
| `model`          | string           | —       | Falls back to `config.defaultModel`                |
| `variant`        | string           | —       | opencode model variant                             |
| `temperature`    | number `[0, 2]`  | —       |                                                    |
| `steps`          | int > 0          | —       | opencode step budget                               |
| `maxSteps`       | int > 0          | —       |                                                    |
| `skills`         | `string[]`       | `[]`    | Names of skills already registered in /skills      |
| `permission`     | `object`         | `{}`    | Passed verbatim into opencode config               |
| `frontmatterExtra` | `object`       | `{}`    | Any frontmatter key not above; round-trips         |

## Body markdown

Everything below the `---` frontmatter delimiter is the agent's system prompt.
The framework appends a small **English protocol block** to the user prompt
at run time, telling the model to wrap its final answer in:

```xml
<workflow-output>
  <port name="findings">…</port>
  <port name="summary">…</port>
</workflow-output>
```

If multiple envelopes appear in stdout, the **last one wins**. Missing
declared ports are stored as empty strings. Ports the agent emits but did
not declare are logged with a warning and dropped.

## Concurrency

The platform has four semaphores:

| Semaphore          | Capacity                                     | Who holds it                                                                   |
| ------------------ | -------------------------------------------- | ------------------------------------------------------------------------------ |
| Agent pool         | `config.maxConcurrentNodes` (default 4)      | Agent nodes, workgroup host nodes, and each fan-out shard/aggregator            |
| Script pool        | `config.maxConcurrentScriptNodes` (default 4)| RFC-253 script nodes — a **separate** daemon pool (RFC-266)                     |
| Per-task write     | 1                                            | Every node, but only across snapshot-at-dispatch and merge-back                 |
| Fan-out sub-pool   | `config.multiProcessSubprocessConcurrency`   | Each shard of one task's fan-out, **inside** its agent-pool slot                |

Notes:

- The two daemon pools are **fully independent**: a script node never queues
  behind agent runs and vice versa, so peak child processes are the sum of both
  caps.
- Taking **no** slot: `call-workflow` / `call-workgroup` nodes (the child task's
  own nodes compete instead), the RFC-130 merge-conflict agent (bypasses the
  pool to avoid a lock cycle), and wrapper containers themselves.
- Effective fan-out parallelism is `min(free agent slots, sub-pool capacity)`.
- All four caps are hot-applied when settings are saved — running tasks and
  nodes already queued for a slot included (RFC-266).
- RFC-130 superseded the old "writers serialize per task" model: every node runs
  in its **own isolated worktree** and merges its delta back, so agent-level
  `readonly` no longer partitions the scheduler. What `readonly` still governs
  is the filesystem boundary a node runs under, and for script nodes it also
  decides whether the node runs in place against the canonical worktree (with a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangbinquan/agent-workflow](https://github.com/wangbinquan/agent-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
