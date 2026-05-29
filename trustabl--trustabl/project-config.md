---
trigger: always_on
description: This file captures durable architectural commitments that span the whole
---

# Instructions for Claude — Trustabl

This file captures durable architectural commitments that span the whole
codebase. Per-area conventions live in nested CLAUDE.md files (see
[`testdata/rules-fixture/CLAUDE.md`](testdata/rules-fixture/CLAUDE.md)
for rule authoring). The detection rule packs do **not** live in this repo:
they live in the external **`trustabl-rules`** repository
(`https://github.com/trustabl/trustabl-rules`), which the engine pulls at
scan time. `testdata/rules-fixture/` is an in-engine **test mirror** of those
packs — see [Two-repo rule model](#two-repo-rule-model-rules-vs-engine) below,
which is required reading before touching any rule.

For the current implementation, see [`ARCHITECTURE.md`](ARCHITECTURE.md).
This file is for principles; ARCHITECTURE.md is for facts.

## Project naming

The product is named **Trustabl** (capital T). Use this spelling in all
human-facing prose: docs, status reports, finding messages shown in scan
reports, and CLI help text.

The lowercase `trustabl` is reserved for machine identifiers that must
**not** be capitalized: the binary name, the CLI command (`trustabl scan`,
`trustabl version`), the Go module path (`github.com/trustabl/trustabl`),
and internal prefixes (e.g. the clone temp-dir `trustabl-clone-*`). When in
doubt: if a human reads it as a sentence, it's "Trustabl"; if a machine
parses it as a token, it's `trustabl`.

## Detection model: four scopes

Every rule is classified into exactly one of four scopes. The `scope:`
field on a rule is REQUIRED for new rules; legacy rules without it default
to `tool` (the historical behavior).

- **`tool`** — fires per tool definition.
  - **Input**: a `ToolDef` — discovery produces these from a
    `@function_tool`-decorated function (`openai_tool`), a `@tool` /
    `@claude_tool` / `claude_agent_sdk` function (`claude_sdk_tool`), the
    Claude TS SDK `tool(...)` factory call (also `claude_sdk_tool`), a
    `@server.tool` / `@mcp.tool` / `.register_tool` MCP registration
    (`mcp_tool`), or a bare shell-invoking function (`shell_invocation`) —
    plus its parsed file. (Hosted-tool instances like `WebSearchTool()` are
    captured as `HostedToolDef`, not `ToolDef`, and are agent-scope edge data,
    not tool-scope inputs.)
  - **Examples**: missing docstring, network call without timeout, untyped
    params, unnormalized path in `open()`.

- **`agent`** — fires per agent declaration.
  - **Input**: an `AgentDef` — a single `Agent(...)` /
    `SandboxAgent(...)` / Claude Python `AgentDefinition(...)` call, a
    Claude TS typed-const `AgentDefinition`, a Claude TS sub-agent inline
    in `options.agents`, or the TS `query(...)` main-thread agent
    (`QueryMainAgent`) — with all its kwargs captured and edges to its
    tools / handoffs / guardrails resolved.
  - **Examples**: agent has no `input_guardrails`,
    `tool_use_behavior="stop_on_first_tool"` paired with
    filesystem-touching tools, handoff to subagent that has fewer
    guardrails than the parent.

- **`subagent`** — fires per `.claude/agents/*.md` declaration.
  - **Input**: a `SubagentDef` parsed from markdown frontmatter (`name`,
    `description`, `tools[]`, `model`). Matched at any path depth
    (monorepo-safe). Carries no `Language` field — markdown frontmatter is
    language-agnostic, so subagent rules carry no `language:` field either
    and the detector does not gate on language.
  - **Examples**: subagent granted `Bash` despite a read-only description
    (CSDK-110), description-vs-tools mismatch, no `name`.

- **`repo`** — fires once per scan against the whole repo.
  - **Input**: `RepoProfile` + `RepoInventory` (languages, declared SDK
    deps, the `ScanManifest` file inventory and discovered components, plus
    the discovered tools/agents and `SDKsDetected`).
  - **Examples**: project-wide tracing config has no custom processor;
    no `SandboxAgent` anywhere in a project that ships FS-touching tools.

What older code calls `singleton: true` is `repo` scope in disguise.
Promote to explicit `scope: repo` when touching those rules.

## Scanning pipeline

The scan is a flat sequence of steps; the output of each is the typed
input to the next. The boundary between the cheap recon step and the
AST-driven steps that follow is load-bearing: recon stays cheap so it can
gate whether the expensive AST work runs at all, and the inventory those
steps build is what makes policy selection data-driven rather than
statically configured.

Before the pipeline runs, the CLI resolves detection rules from the
external `trustabl-rules` git repository (`rulesource.DefaultRepoURL`,
currently `https://github.com/trustabl/trustabl-rules`; the engine embeds
none — see `internal/rulesource/`) and hands them to `scanner.Run` as an
`fs.FS`. The
resolution path fetches the configured ref, caches the clone under
`os.UserCacheDir()/trustabl/rules/<sha>/`, falls back to the cache when the
network is unreachable, and gates the pack's `manifest.yaml`
`schema_version` against the engine's `rules.SupportedSchemaVersion`. No
usable rules (none cached and none fetchable, or none compatible) is a
hard exit 2 — the engine never runs rule-less. The resolved rules SHA is
recorded on `ScanResult` and folded into `ScanID`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trustabl/trustabl](https://github.com/trustabl/trustabl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
