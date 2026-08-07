---
trigger: always_on
description: This document tells an LLM agent how to drive the `tabstack` CLI correctly. It is
---

# Using `tabstack` from an AI agent

This document tells an LLM agent how to drive the `tabstack` CLI correctly. It is
a reference, not a tutorial: every command, flag, input format, output shape,
and failure mode is listed so you can call the tool without guessing.

## TL;DR for agents

- Always pass **`-o json`** so output is deterministic and parseable. Without it,
  output is pretty-printed on a TTY and only switches to JSON when piped.
- Provide an API key via the **`TABSTACK_API_KEY`** env var (preferred for
  automation) or `--api-key`. Do not rely on an interactive `auth login` prompt.
- **Branch on the exit code**, not on stderr text: `0` ok, `1` runtime/network,
  `2` bad input or missing key, `3` API error or task failure.
- `extract json` / `generate json` return **exactly the JSON shape your schema
  describes**. Nothing is wrapped or reshaped. Validate your schema before
  calling; malformed schemas fail locally with exit `2`.
- `automate` and `research` **stream**. In `-o json` they emit **NDJSON** (one
  JSON object per line); read line by line, do not `JSON.parse` the whole output.

## Setup

```bash
export TABSTACK_API_KEY="<key>"        # preferred for non-interactive use
# optional:
export TABSTACK_BASE_URL="<url>"       # override API root
```

Key resolution precedence (highest first): `--api-key` flag → `TABSTACK_API_KEY`
→ config file (`~/.config/tabstack/config.toml`). If no key is found, commands
that hit the API exit `2` (non-retryable config error) with a clear message.

## Global flags (valid on every command)

| Flag | Effect |
|------|--------|
| `-o, --output pretty\|json` | **Set `json` for agents.** Default auto-detects (pretty on TTY, json when piped). |
| `--api-key <key>` | API key, overrides env + config. |
| `--base-url <url>` | API root, overrides env + config. |
| `--no-color` | Disable ANSI colour (also honours `NO_COLOR`). Irrelevant under `-o json`. |
| `--timeout <dur>` | Timeout for **non-streaming** calls only, e.g. `30s`, `2m`. Ignored by `automate`/`research`. |

## Input value convention

`--schema`, `--instructions`, and `--data` each accept one of three forms:

- a **literal string**: `--schema '{"type":"object"}'`
- **`@file`**: `--schema @schema.json` (reads the file)
- **`-`**: `--schema -` (reads stdin; only one flag per call may use `-`)

JSON-valued flags are validated locally before the request; invalid JSON fails
with exit `2` and no network call.

## Commands

### `extract markdown <url>`: page → clean Markdown

Non-streaming. Single JSON response.

| Flag | Required | Notes |
|------|----------|-------|
| `--effort min\|standard\|max` | no | Fetch effort, default `standard`. See table below. |
| `--geo <CC>` | no | ISO 3166-1 alpha-2 country, e.g. `GB`. |
| `--metadata` | no | Include page metadata (title, author, …) in the response. |
| `--nocache` | no | Bypass cache, fetch fresh. |

Output (`-o json`):
```json
{"content":"# Title…","url":"https://…","metadata":{"title":"…","author":"…"}}
```
`metadata` is present only when `--metadata` was passed.

Example:
```bash
tabstack -o json extract markdown https://example.com --metadata
```

### `extract json <url> --schema …`: page → schema-shaped JSON

Non-streaming. **The response is exactly your schema's shape**, returned verbatim.

| Flag | Required | Notes |
|------|----------|-------|
| `--schema` | **yes** | JSON schema (literal / `@file` / `-`). Must be valid JSON. |
| `--effort` / `--geo` / `--nocache` | no | As above. |

Example:
```bash
tabstack -o json extract json https://example.com \
  --schema '{"type":"object","properties":{"title":{"type":"string"}}}'
```

### `generate json <url> --instructions … --schema …`: fetch + AI transform → schema-shaped JSON

Non-streaming. Fetches the page, then transforms its content with AI per your
instructions into the schema shape. Response is your schema's shape, verbatim.

| Flag | Required | Notes |
|------|----------|-------|
| `--instructions` | **yes** | Transform prompt (literal / `@file` / `-`). Max **20,000** chars (validated locally). |
| `--schema` | **yes** | Output JSON schema (literal / `@file` / `-`). |
| `--effort` / `--geo` / `--nocache` | no | As above. |

Constraint: `--instructions` and `--schema` cannot **both** read from `-` (stdin)
in one call.

Example:
```bash
tabstack -o json generate json https://example.com \
  --instructions "Extract the headline and a one-sentence summary." \
  --schema @out-schema.json
```

### `agent automate <task>`: natural-language browser automation (streaming)

Runs server-side and **streams events**. The task description is the positional
argument.

| Flag | Required | Notes |
|------|----------|-------|
| `--url <url>` | no | Starting URL for the task. |
| `--data <json>` | no | JSON context object (literal / `@file` / `-`), e.g. form values. |
| `--guardrails <text>` | no | Safety constraints, e.g. "read-only, do not submit forms". |
| `--max-iterations <n>` | no | 1–100 (validated locally). |
| `--max-validation-attempts <n>` | no | 1–10 (validated locally). |
| `--geo <CC>` | no | Geotarget country code. |
| `--interactive` | no | Allow the task to **pause and request input** mid-run (see `agent input`). |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mozilla-Ocho/tabstack-cli](https://github.com/Mozilla-Ocho/tabstack-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
