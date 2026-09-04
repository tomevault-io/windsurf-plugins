---
trigger: always_on
description: When adding or changing a service, custom protocol, transform, credential pool, async workflow, Pack documentation, release, or rollback, load `.agents/skills/localrouter-protocol-pack/SKILL.md` first and follow its complete lifecycle.
---

# LocalRouter Agent instructions

When adding or changing a service, custom protocol, transform, credential pool, async workflow, Pack documentation, release, or rollback, load `.agents/skills/localrouter-protocol-pack/SKILL.md` first and follow its complete lifecycle.

Consumer Agents discover capabilities from `http://127.0.0.1:8317/.well-known/localrouter.json`; they must not depend on repository filesystem access. Start with `lr tree [pack]`: every callable service appears as either a lightweight compatibility Pack (`/v1`, `/v1beta`) or a full Protocol Pack (`/p/<pack>`), while `/w` and `/mcp` are Pack projections rather than separate service definitions. Classify what is being searched before narrowing it: use `lr find operation <intent>` for callable Protocol Pack operations, `lr find model <name>` for live upstream model discovery, and `lr find pool <provider-or-pack>` for readiness, quota and pricing. Final model selection must use `lr find model --exact <pack>:<model-id>` and return one exact live result. Model and catalog output is capped at 20 by default; refine first and use `--all` only when the complete machine catalog is required. `lr find <mixed intent>` returns all three domains separately; it never treats an OMP/runtime configuration as a LocalRouter operation. Use `lr catalog [pack]` or `lr catalog --all` for independently addressable Protocol Pack operations, or `lr resolve <capability>` for exact/fuzzy operation matches, then `lr compare <pack.operation>...` when multiple candidates remain. LocalRouter never merges providers or silently chooses one: compare every returned `operation_key`, Pack, readiness, verification level, pool, schema, pricing, retry and guide, choose explicitly, then use `lr describe` and `lr preflight` before a paid or side-effecting call. Treat a nonzero `lr preflight` exit as blocking and consume its structured `next_action` and `alternatives`. Reuse cached contracts only while both `contract.digest` and `contract.schema_version` are unchanged.

Treat `operation_key` and `operation_id` as semantic selectors, never as URL fragments. When an `lr` command already takes a Pack, its operation argument accepts either the bare `operation_id` or the published Pack-qualified `operation_key`, so an Agent may safely paste a catalog result. Direct HTTP calls must use the exact published `call_url`; prefer `lr call`, Agent describe/preflight, or MCP so dotted IDs such as `chat.completions` cannot be mistaken for slash paths.

Treat `request_example` as shape-only. Resolve every published `dynamic_inputs` field before calling; for `model`, use `lr find model --exact <pack>:<model-id>` or the advertised models operation and choose an exact current `data[].id` instead of assuming the example value is available. Preflight does not call the provider catalogue and therefore cannot prove that a guessed model exists. A model search result is provider-qualified as `<pack>:<model-id>` and includes `compatible_operations`, derived from the Pack's declared dynamic-input relationship rather than a hard-coded API family. Select both explicitly, then configure OMP or another external Agent runtime separately if needed.

Use `lr run` for an operation or persistent workflow. For long workflows, retain the returned Pack, workflow and Job ID, then use `lr watch`; it has no default timeout and can be resumed with the same Job ID. Use `lr cancel` only when the workflow advertises cancellation. Treat structured `code`, `retryable`, `retry_after`, `next_action`, and `alternatives` as authoritative; never blindly replay an unknown side-effecting outcome.

For an authorized real or paid call, invoke `lr call` exactly once and capture its raw response plus exit status before running `jq` or another parser. Summarize that captured value offline. A parser, display, or pipe failure is not authority to repeat the upstream request.

Consumer API Tokens are long-lived and unlimited by default, but remain call-only. This never bypasses Pack pool concurrency, lease expiry, cooldown, health, or quota eligibility. Maintenance is separate and uses the administrator credential by default. Optional Agent maintenance is disabled by default; an Agent may call `/manage/mcp` only when discovery reports it enabled and the human supplied a distinct maintenance-only Token with `localrouter.maintain`. Agents must not request or read the administrator credential.

The human browser console and `/local/api` use password-free loopback access by default and may be protected with a custom password from Run Overview. This convenience does not authorize Agent mutation: Agents still use the explicit `/manage/mcp` maintenance lane and must not treat an open console as delegated authority.

Keep installed secrets below `$XDG_DATA_HOME/localrouter/` with mode `0600`; isolated tests may override `LOCAL_GATEWAY_DATA_DIR`. Never put credentials, cookies, pool contents, or private upstream addresses in source, guides, logs, test output, or `.ai` project-visible notes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vimalinx/LocalRouter](https://github.com/vimalinx/LocalRouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
