---
trigger: always_on
description: `bydbctl agent` is a two-pane terminal workspace where Codex or Claude holds a multi-turn BanyanDB conversation, discovers schemas, proposes typed query
---

# BYDBQL Agent TUI

`bydbctl agent` is a two-pane terminal workspace where Codex or Claude holds a multi-turn BanyanDB conversation, discovers schemas, proposes typed query
plans, and safely runs read-only queries.

Install Codex CLI 0.144.5 or newer and log in before starting the TUI:

```shell
codex login
```

Codex owns its login credentials. bydbctl neither reads nor copies them.

To use Claude instead, install Claude Code and authenticate it using either its normal login flow or `ANTHROPIC_API_KEY`:

```shell
claude auth login
bydbctl agent --provider claude
```

Claude Code owns its login credentials. bydbctl starts the CLI directly; it does not call the Anthropic Messages API or embed the TypeScript/Python Claude Agent SDK.

## Start

```shell
bydbctl agent \
  --addr http://localhost:17913 \
  --goal "top slow payment endpoints in the last 30 minutes"
```

To use a Codex binary outside `PATH`:

```shell
bydbctl agent \
  --codex-command /path/to/codex \
  --addr https://banyandb.example:17913 \
  --enable-tls \
  --cert /path/to/ca.pem
```

To select Claude Code or use a binary outside `PATH`:

```shell
bydbctl agent \
  --provider claude \
  --claude-command /path/to/claude \
  --claude-model sonnet \
  --claude-max-turns 12 \
  --addr https://banyandb.example:17913
```

`--claude-api-key` and `--claude-base-url` optionally override `ANTHROPIC_API_KEY` and `ANTHROPIC_BASE_URL` for the child process.
When omitted, Claude Code uses its normal authentication and provider configuration.

The Agent TUI uses the same `--addr`, username/password, TLS certificate, and `--insecure` semantics as the normal bydbctl HTTP commands. Codex never
receives those settings or BanyanDB credentials.

## Controlled tools and safety

Each TUI session creates a private, local MCP bridge. It exposes exactly these tools:

- `list_groups_schemas`
- `describe_schema`
- `propose_query_plan`
- `validate_bydbql`
- `execute_bydbql`

A schema, capability, or usage question is answered from `list_groups_schemas` and `describe_schema` alone. Those turns are classified as
answer turns, and the provider is explicitly instructed not to compile a plan or read stored rows. Asking what fields a resource has therefore
inspects the schema instead of querying data. A request for stored data follows the full query workflow below.

When such a question names exactly one resource, bydbctl reads that schema itself and never starts a provider turn at all; see
[Direct schema lookups](#direct-schema-lookups). The reachable data is identical either way, because both paths call the same read-only schema API.

The Agent starts with no selected schema. It ranks catalog candidates but resolves resources against the complete discovered catalog using exact type,
name, and group identity. It never silently substitutes a similar resource or another time granularity. Typed schemas are cached per resource and group set,
so a workflow can compile several resources independently. If the best choices remain ambiguous, it asks one focused clarification question. The Schema
panel is read-only and cannot pin a resource.

`propose_query_plan` accepts a strict JSON plan or a bounded workflow. The bridge loads the exact schema when needed, binds the compiled query to a
schema fingerprint, and returns path-based diagnostics with allowed values when compilation fails. The planner supports typed projections, tag/entity
comparison and `IN` filters with `AND`/`OR`, exact sortable index-rule ordering, numeric Measure aggregation/grouping, empty Trace projection, and
registered TopN aggregations. A normal Measure is never treated as a TopN aggregation. Failed proposals remain visible diagnostics but are not
executable candidates.

The planner rejects unknown JSON fields, implicit value coercion, field filters, tag aggregation, invalid time formats, out-of-range limits, `MATCH`,
`HAVING`, `OFFSET`, `STAGES`, `WITH QUERY_TRACE`, joins, and unknown columns rather than guessing. `validate_bydbql` remains a parse/safety and
manual-editor check; only a successful `propose_query_plan` can publish a provider candidate. The bridge rejects every other tool, shell command,
external MCP server, dynamic registration, and download.

When `propose_query_plan` returns `valid=false`, the provider receives the structured diagnostic and repairs the plan within the same agent turn.
The bridge allows at most three proposal attempts per schema-description cycle and reports the exhausted repair budget instead of looping indefinitely.

For Codex, bydbctl starts one isolated `codex app-server --stdio` process with an ephemeral in-memory thread, read-only sandboxing, and no approval requests. Built-in
shell, web, app, plugin, hook, sub-agent, goal, memory, and shell-snapshot features are disabled. Existing user MCP servers are disabled for this process.
Startup fails unless runtime inventory contains exactly the five controlled tools and no uncontrolled tools or resources.

For Claude, bydbctl starts `claude --print --output-format stream-json` directly from Go. Each TUI turn gets a supervised CLI process; later turns use

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/skywalking-banyandb](https://github.com/apache/skywalking-banyandb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
