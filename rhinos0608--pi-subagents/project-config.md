---
trigger: always_on
description: An agent is a markdown file: YAML frontmatter on top, a system prompt below. The frontmatter defines the specialist that runs as the child session.
---

# Agents

An agent is a markdown file: YAML frontmatter on top, a system prompt below. The frontmatter defines the specialist that runs as the child session.

```yaml
---
name: scout
description: Fast codebase recon
tools: read, grep, find, ls
---

Your system prompt goes here.
```

## Where agents live

Lowest to highest priority:

| Scope | Path |
|-------|------|
| Builtin | `~/.pi/agent/extensions/subagent/agents/` |
| Installed package | `package.json` `pi-subagents.agents` or `pi.subagents.agents` |
| User | `~/.pi/agent/agents/**/*.md` |
| Project | Project config `agents/**/*.md` (`.pi/agents/**/*.md` in standard Pi) |

Discovery notes:

- Project discovery also reads legacy `.agents/**/*.md` files. If both `.agents/` and the project config agents directory define the same parsed runtime agent name, the project config directory wins.
- Nested subdirectories are discovered recursively. `.chain.md` files do not define agents.
- User and project settings can add extra recursive scan roots with `subagents.agentScanDirs`; fixed user/project agent directories keep higher priority than same-name agents from scan roots.
- Use `subagents.agentExcludeDirs` to prune literal directory subtrees without disabling legacy agents. See [configuration.md](configuration.md#excluded-agent-directories-settings) for path resolution, scope, and exemptions.
- Installed Pi packages can expose agent directories from either `{"pi-subagents":{"agents":["./agents"]}}` or `{"pi":{"subagents":{"agents":["./agents"]}}}` in their package manifest. Package agents load above builtins and below user/project agents.
- Use `agentScope: "user" | "project" | "both"` to control discovery. `both` is the default, and project definitions win runtime-name collisions.

## Builtin agents

Builtins load at the lowest priority, so a user or project agent with the same name overrides them. They do not pin a provider model; they inherit your current Pi default model unless you set `subagents.defaultModel` or `subagents.agentOverrides.<name>.model` (see [models.md](models.md)).

| Agent | Use it when you want... |
|-------|--------------------------|
| `scout` | Fast local codebase recon: relevant files, entry points, data flow, risks, and where another agent should start. |
| `researcher` | Web/docs research with sources: official docs, specs, benchmarks, recent changes, and a concise research brief. |
| `evidence-auditor` | Independent evidence review of important claims in an existing research brief. |
| `worker` | Implementation work, including approved oracle handoffs. It edits files, validates, and escalates unapproved decisions instead of guessing. |
| `reviewer` | Code review and small fixes. It checks the implementation against the task/plan, tests, edge cases, and simplicity. |
| `oracle` | A second opinion before acting. It challenges assumptions, catches drift, and recommends the safest next move without editing. |
| `delegate` | A lightweight general delegate when you want a child agent that behaves close to the parent session. |

Rule of thumb: `scout` before you understand the code, `researcher` before you trust external facts, `evidence-auditor` before you rely on important research, `worker` to implement, `reviewer` to check, and `oracle` when the decision itself feels risky.

`oracle` is an advisory reviewer that critiques direction and proposes an execution prompt without editing files. `advisor` is the same bundled role under the Claude Code-compatible name.

### Optional Surf integration

When `surf-cli` is installed and loaded, Surf can expose a `gpt-pro` package agent through the `surf-oracle` external-job provider. It starts through the same `subagent({ agent: "gpt-pro" })` mental model as any other agent, but Surf owns the package agent and provider. Surf maps `model: pro` to ChatGPT GPT-5.6 Sol Pro web mode. pi-subagents does not own that model mapping.

If you disabled the old bundled `gpt-pro` workaround with `agentOverrides.gpt-pro.disabled`, remove that override before using Surf's package agent.

The Pi async run remains the source of truth for status, artifacts, wake/wait, mission attachment, retention, and diagnostics.

### Command-runner agents as typed steps

`runner.type: external-cli` with a plain `command` (no `adapter`) runs any local executable as a subagent: the assembled prompt is written to stdin, stdout becomes the child's output, and the run gets the usual run id, status, mission entry, and workflow key. This is how a classifier, a scoring script, or a small evaluation model becomes a `runs.run` step. Such agents are async-only (a per-call `async` override is rejected on model calls), receive no forked transcript, and cannot produce `structuredOutput` themselves; parse their `output` in the workflow, or pair them with a [typed gate](tool-reference.md#typed-gates). Generic commands are local-only; saved-machine placement accepts only the code-owned adapters below.

### Advisory runner data boundary


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhinos0608/Pi-Subagents](https://github.com/rhinos0608/Pi-Subagents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
