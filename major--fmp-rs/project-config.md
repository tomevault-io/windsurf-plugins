---
trigger: always_on
description: `fmp-agent` is an unofficial CLI for the Financial Modeling Prep stable API, optimized for predictable JSON output in shell pipelines and LLM tool-calling. It is not affiliated with, endorsed by, or sponsored by Financial Modeling Prep.
---

# fmp-agent CLI reference

`fmp-agent` is an unofficial CLI for the Financial Modeling Prep stable API, optimized for predictable JSON output in shell pipelines and LLM tool-calling. It is not affiliated with, endorsed by, or sponsored by Financial Modeling Prep.

This document is the long-form reference. For programmatic discovery, prefer the `schema` subcommand below.

## 1. Self-discovery (start here)

LLMs and tool runners should discover the command surface from the binary itself instead of parsing this document:

```bash
fmp-agent doctor             # JSON readiness: version, base URL validity, API key presence
fmp-agent commands           # list all leaf command paths, one per line
fmp-agent completions <shell> # generate bash/zsh/fish/powershell completions script
fmp-agent schema             # versioned JSON: { schema_version, binary, version, commands[] }
fmp-agent help environment   # environment variables, .env, and config precedence
fmp-agent help exit-codes    # exit codes and structured stderr errors
fmp-agent help schema        # schema and tool-calling guidance
fmp-agent help examples      # representative command examples
fmp-agent --help             # human-readable top-level help
fmp-agent <GROUP> --help     # human-readable per-group help
fmp-agent <GROUP> <CMD> --help  # human-readable per-command help, includes Examples
```

`fmp-agent doctor`, `fmp-agent commands`, `fmp-agent completions`, `fmp-agent schema`, and `fmp-agent help <topic>`:

- Do **not** require `FMP_API_KEY` and make **no** network requests.
- `doctor` emits JSON with `ok`, `version`, sanitized `base_url`, `api_key.configured`, and `live_connectivity.checked: false` so callers can verify local readiness without consuming quota.
- `commands` prints one leaf path per line, sorted alphabetically (e.g. `analyst grades`, `company profile`).
- `completions` generates a shell completions script (bash/zsh/fish/powershell) on stdout.
- `schema` emits `schema_version: 3` today; the shape is experimental and may change between releases.
- `help <topic>` prints operational guidance from the installed binary. Topics are `environment`, `exit-codes`, `schema`, and `examples`.
- Each schema command entry has `name`, `path`, `aliases`, `preferred_path`, `api_key_required`, `about`, `long_about`, and `args`.
- Each arg has `name`, `kind`, `required`, `default`, `value_name`, `help`, `long` (exact `--flag` spelling or null for positional), `short` (single-char flag or null), `parser` (type hint: `string`, `integer`, `bool`, `enum`, or `count`), `possible_values` (array of `{"name": "...", "help": "..."}` for enum args, null otherwise), and `multi_value` (whether the arg accepts repeat values).
- Use these as the source of truth for available commands and arguments; treat the catalog in section 6 as a curated index.

Top-level aliases (no API key required for discovery):

```bash
fmp-agent quote AAPL         # alias for market quote
fmp-agent historical AAPL    # alias for market historical
fmp-agent profile AAPL       # alias for company profile
fmp-agent earnings           # alias for calendar earnings
```

## 2. Install

```bash
cargo install rusty-fmp --locked
```

GitHub releases also publish cargo-dist archives and shell or PowerShell installers. The Cargo package is `rusty-fmp`; the installed binary is `fmp-agent`. From a repo checkout, substitute `cargo run -- <GROUP> <CMD>` for `fmp-agent <GROUP> <CMD>`.

## 3. Invocation contract

```bash
fmp-agent [OPTIONS] <GROUP> <CMD> [ARGS]
```

Stable behavior callers can rely on:

- **Success**: the raw FMP JSON payload on **one line** to stdout, exit code 0.
- **Runtime error** (config, network, API, parse, strict empty result): JSON envelope on **stderr**, non-zero exit:
  `{"ok": false, "error": {"kind": "...", "message": "..."}}`
- **Rate limit**: HTTP 429 exits 5 with `error.kind` set to `rate_limited`. Retry later with backoff instead of treating it as a subscription, authentication, or generic API failure.
- **Parse error** (bad flags or missing required args): Clap's human-readable usage text on stderr, exit code 2. To distinguish programmatically, check exit code first; only parse stderr as JSON for codes 3-7.
- Empty JSON arrays are successful raw FMP responses by default because they can be valid for date ranges, news windows, or unknown symbols. For symbol lookups where an empty result should stop automation, pass `--strict-empty`; it exits 7 with `empty_result` and suggests `fmp-agent search <SYMBOL>`.
- Help and version output are human-readable text, not JSON.
- The CLI deliberately offers no output formatting, filtering, or pagination flags. Pipe through `jq` for selection.
- Running `fmp-agent` with no command prints help and exits.

### Exit codes

| Code | Meaning                                                        |
| ---- | -------------------------------------------------------------- |
| 0    | Success                                                        |
| 2    | Usage error (Clap parse failure)                               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [major/fmp-rs](https://github.com/major/fmp-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
