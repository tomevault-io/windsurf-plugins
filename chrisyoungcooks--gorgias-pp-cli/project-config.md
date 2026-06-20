---
trigger: always_on
description: Every Gorgias support workflow, agent-native, in one binary. Trigger phrases: `list gorgias tickets`, `search gorgias`, `find a ticket about <topic>`, `show gorgias customers`, `what tickets came in yesterday`, `tag this gorgias ticket`, `reply to gorgias ticket`.
---


# Gorgias — Printing Press CLI

## Prerequisites: Install the CLI

This skill drives the `gorgias-pp-cli` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

```bash
go install github.com/chrisyoungcooks/gorgias-pp-cli/cmd/gorgias-pp-cli@latest
go install github.com/chrisyoungcooks/gorgias-pp-cli/cmd/gorgias-pp-mcp@latest
```

Requires Go 1.26.3 or newer. Verify with `gorgias-pp-cli --version`. Ensure `$GOPATH/bin` (or `$HOME/go/bin`) is on `$PATH`. If `--version` reports "command not found", the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

A token-efficient CLI for the Gorgias customer support API with a sibling MCP server. Covers 108 endpoints across tickets, customers, messages, macros, tags, teams, integrations, events, rules, satisfaction surveys, voice calls, custom fields, and views. Built for AI agents first: JSON output, structured doctor checks, local SQLite mirror, and code-orchestration MCP — 15 tools (~9K tokens measured for `tools/list`) cover the full 108-endpoint surface via the `gorgias_search` + `gorgias_execute` gateway.

## When to Use This CLI

Reach for gorgias-pp-cli whenever a support workflow needs to read or write Gorgias data: pulling recent tickets for digest, searching past conversations for context, automating bulk tag/macro applications, monitoring oncall queues, or building agent-driven escalation flows. The sibling MCP server lets LLMs drive the same surface without shelling out.

## Unique Capabilities

These capabilities aren't available in any other tool for this API.
- **`gorgias-pp-cli doctor --json`** — Probes /account with the configured credentials and reports `credentials: valid` only when an authenticated call succeeds.

  _Saves the first-five-minutes credential-debug cycle when wiring up an agent._

  ```bash
  gorgias-pp-cli doctor --json
  ```
- **`gorgias-pp-cli sync --resources tickets --since 7d && gorgias-pp-cli search 'refund' --agent`** — Syncs API data to a local SQLite DB so subsequent searches, analytics, and joins run without hitting the API.

  _Makes repeated agent-driven lookups (e.g. searching for similar past tickets) practical at scale. Ticket `--since` uses documented `order_by=updated_datetime:desc` plus local filtering; do not add undocumented filters like `updated_datetime__gte` unless Gorgias documents them and a live smoke confirms them._

  ```bash
  gorgias-pp-cli sync --resources tickets --since 30d --json
  ```

## Command Reference

**account** — Account-level settings and tenant metadata

- `gorgias-pp-cli account get` — Retrieve the current Gorgias account's metadata: subdomain, plan, billing state, and account-level flags. Use this...
- `gorgias-pp-cli account settings-create` — Create a new account-level settings record for the current Gorgias tenant. Use when bootstrapping a fresh tenant or...
- `gorgias-pp-cli account settings-list` — List the global settings on the current Gorgias account (business hours, language, default channels, notification...
- `gorgias-pp-cli account settings-update` — Update an account settings record by `id`. Use this to flip a tenant-wide flag, change business hours, or adjust a...

**custom-fields** — Define and manage custom fields on tickets and customers

- `gorgias-pp-cli custom-fields create` — Define a new custom field on tickets or customers (the only supported `object_type` values). Required body:...
- `gorgias-pp-cli custom-fields get` — Fetch a single custom field definition by `id`, returning its data type, label, target object, and option list. Use...
- `gorgias-pp-cli custom-fields list` — List custom field definitions for a single `object_type` (`Ticket` or `Customer` — REQUIRED query param)....
- `gorgias-pp-cli custom-fields update` — Update one custom field definition by `id` — relabel it, change its options, or toggle visibility. Note: this...
- `gorgias-pp-cli custom-fields update-all` — Bulk-update multiple custom field definitions in one call (no path id). Useful when reordering picklist options or...

**customers** — Read and write Gorgias customer records (CRM core)

- `gorgias-pp-cli customers create` — Create a new customer record. Pass `name`, `email`, optional `channels` (email/phone/social handles), and `data` for...
- `gorgias-pp-cli customers custom-fields-list` — List every custom field value attached to a single customer (`id`). Use to read CRM-style attributes (lifetime...
- `gorgias-pp-cli customers custom-fields-set` — Set a single custom field value on a customer: first `{id}` is the customer, second `{id}` is the custom field. Use...
- `gorgias-pp-cli customers custom-fields-set-all` — Bulk-set custom field values on a single customer (`id`) — pass an array of field/value pairs. Preferred over the...
- `gorgias-pp-cli customers custom-fields-unset` — Clear a custom field value on a customer: first `{id}` is the customer ID, second `{id}` is the custom field ID. Use...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisyoungcooks/gorgias-pp-cli](https://github.com/chrisyoungcooks/gorgias-pp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
