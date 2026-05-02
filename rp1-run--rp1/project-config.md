---
trigger: always_on
description: This file is intentionally narrow. Use it for agent and skill authoring rules that are specific to rp1.
---

# rp1 Agent Authoring Guide

This file is intentionally narrow. Use it for agent and skill authoring rules that are specific to rp1.

1. Read `.rp1/context/index.md` first.
2. Then load only the KB files needed for the task.

Project understanding belongs in `.rp1/context/`. This file covers runtime and authoring constraints.

## Core Rules

### Namespace prefixes

Use these prefixes exactly:

- Skills: `/rp1-base:skill-name`, `/rp1-dev:skill-name`, `/rp1-utils:skill-name`
- Agent references: `subagent_type: rp1-base:agent-name` for Claude Code, `subagent_type: @rp1-dev/agent-name` for OpenCode

### Subagent limitations

Subagents generally cannot spawn other agents. If an agent is designed to run as a subagent:

- Do not use SlashCommand to call other commands.
- Do not call `/rp1-base:knowledge-load` from inside the subagent.
- Inline only the prompt text or KB guidance the subagent actually needs.

### Codex task shorthand

For Codex agents, interpret any instruction in the form `Task: <sub-agent-name>` as:

- spawn `<sub-agent-name>`

Treat this as an execution directive, not as descriptive text.

### Codex subagent waiting

When Codex agents spawn subagents in this repo:

- Do not assume a subagent failed just because it does not answer quickly.
- Use longer wait windows for artifact-producing or workflow-heavy subagents.
- Prefer waiting for the spawned subagent to complete before declaring it stalled or rerouting around it, unless the user explicitly wants parallel speculative work.
- Check for expected side effects such as artifact files before concluding the subagent is stuck.
- If the subagent is on the critical path, babysit it with patient polling instead of replacing it after a short timeout.

### Cross-plugin dependency rule

- Dev agents may depend on base.
- Base agents must not call dev commands.
- If a dev workflow needs `/rp1-base:knowledge-load` and it is unavailable, tell the user to install `rp1-base`.

## Parameters and templating

### Argument style

Define parameters using structured `arguments` arrays in frontmatter. Skills nest arguments under `metadata`; agents place them at the top level.

**Skills** (`metadata.arguments`):

```yaml
metadata:
  arguments:
    - name: FEATURE_ID
      type: string
      required: true
      description: "Feature identifier"
    - name: AFK
      type: boolean
      required: false
      default: false
      description: "Non-interactive mode"
      aliases:
        - "afk"
        - "no prompts"
```

**Agents** (top-level `arguments`):

```yaml
arguments:
  - name: FEATURE_ID
    type: string
    required: true
    description: "Feature identifier"
  - name: CONTEXT
    type: string
    required: false
    description: "Optional context"
```

The build pipeline auto-derives the `argument-hint` string from these definitions. Do not write manual `argument-hint` strings or hand-written `## Parameters` / `## 0. Parameters` tables -- both trigger build errors.

Argument names use UPPER_SNAKE_CASE. Supported types: `string`, `boolean`, `enum`. See [docs/concepts/skill-format.md](docs/concepts/skill-format.md) for the full field reference.

### Canonical variable assignment

The build pipeline automatically injects a `## 0. Resolve Arguments` section into every parameterized skill that declares `metadata.arguments`. This section calls `rp1 agent-tools resolve-args --name rp1-{plugin}:{skill}` to resolve both user-supplied arguments and environment variables, returning structured JSON. Skill authors do **not** write this section — it is generated from frontmatter. See [docs/concepts/skill-format.md](docs/concepts/skill-format.md) for details.

**Agents are excluded** from this requirement -- they receive pre-resolved named parameters from parent skills and do not call `resolve-args` themselves.

#### Directory resolution

All project directories are deterministic from the project root. The `RP1_PROJECT_ROOT`, `RP1_KB_ROOT`, and `RP1_WORK_ROOT` environment variables have been removed. Skills and agents should not declare them in `environment` schemas.

To discover project directories, use `rp1 agent-tools rp1-root-dir` which returns:

- `projectRoot` -- the project root (directory containing `.rp1/project_id`)
- `kbRoot` -- always `<projectRoot>/.rp1/context`
- `workRoot` -- always `<projectRoot>/.rp1/work`

#### Path interpolation

When referencing paths in prompts, use relative paths from the project root:

```markdown
.rp1/context/index.md
.rp1/work/features/{FEATURE_ID}/
```

Do not use `${}` shell parameter expansion in Bash snippets intended for Claude Code.

### Artifact Path Contract

When emitting `artifact_registered` events via `rp1 agent-tools emit`, artifact paths must follow these rules:

- **Always include `storageRoot` explicitly.** Do not rely on implicit defaults.
- **Work artifacts** (`storageRoot: "work_dir"`): Paths must be relative to the work root (`.rp1/work/`) without any prefix. Example: `features/my-feature/design.md` (not `work/features/...`).
- **KB artifacts** (`storageRoot: "project"`): Paths must be relative to the project root. Example: `.rp1/context/index.md`.
- **Absolute paths**: Used as-is regardless of `storageRoot`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rp1-run/rp1](https://github.com/rp1-run/rp1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
