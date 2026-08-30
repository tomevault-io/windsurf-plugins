---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

## What this project is

`sn` is a single-binary Rust CLI wrapping ServiceNow's REST APIs: Table, Change Management, Attachment, CMDB, Import Set, Service Catalog, Identification & Reconciliation, CICD (App Repository, Update Sets, ATF), Aggregate, Performance Analytics, and several undocumented endpoints (schema discovery, the REST API Explorer's catalogue, the two that name the caller). Built for LLM agents: stable JSON on stdout, structured JSON errors on stderr, deterministic exit codes, no interactive surprises unless opted into (`sn init`). Commands that *can* prompt (`sn init`, `sn profile add`) only ever do so on a TTY — with a non-terminal stdin they fail naming the missing flag rather than blocking on a read nobody will answer.

## Build, test, lint

```bash
cargo build                    # dev build
cargo build --release          # release (stripped, LTO)
cargo test --workspace         # unit + integration
cargo test --lib query::       # one module
cargo test --test pagination   # one integration file
cargo clippy --all-targets --all-features -- -D warnings   # must pass before commit
cargo fmt --all -- --check
```

Integration tests use `wiremock` to mock ServiceNow and `assert_cmd` to drive the compiled binary. Tests calling `reqwest::blocking::Client` inside `#[tokio::test]` **must** wrap both client construction and method calls in `tokio::task::spawn_blocking`, or the blocking runtime panics on drop inside an async context.

## Architecture

### Module layout

```
src/
  main.rs           → parse Cli, set verbosity, dispatch, map Error → ExitCode
  lib.rs            → pub mod {amb, body, cli, client, config, error, oauth, observability, output, output_table, query} — register new modules here
  amb.rs            → AMB/Bayeux websocket client (record watchers): channel encoding, handshake, long-poll, TLS
  error.rs          → Error enum (7 variants: Usage/Config/Api/Auth/Transport/BrokenPipe/Instance), NO_HTTP_STATUS, exit_code(), to_stderr_json()
  output.rs         → write_value (batched JSON), write_jsonl_line (one record, flushed), emit_value/emit_error (raw writers)
  output_table.rs   → write_table (renders JSON as a comfy-table columnar view for `--output table`)
  config.rs         → Config/Credentials TOML types, load/save, resolve_profile(); OAuth types (AuthMethod, OAuthConfig, OAuthGrant, TokenSet) + token persistence
  client.rs         → reqwest blocking client (proxy/TLS), Auth enum (Basic/Bearer/None), Paginator iterator
  oauth.rs          → OAuth 2.0 for SSO: PKCE, loopback redirect server, token exchange (authorization_code/refresh/client_credentials), ensure_access_token()
  query.rs          → ListQuery/GetQuery/WriteQuery/DeleteQuery → Vec<(String,String)>
  body.rs           → --data / --field parsing into serde_json::Value
  observability.rs  → global AtomicU8 verbosity, log helpers (set_level called in main; log_request/response/body wired into client.rs)
  cli/
    mod.rs          → Cli struct, GlobalFlags, all Subcommand enums + arg structs
    kernel.rs       → the shared command kernel: connect()/emit(), build_profile/build_client/build_client_with_headers, write_response/unwrap_or_raw/take_field/bool_opt, confirm_destructive/confirm_delete
    args.rs         → shared clap arg vocabulary, flattened into the command modules' arg structs: BodyArgs (--data/--field), WaitArgs (--wait/--wait-timeout), DisplayValueOpt, SetLimit/Paging (const-generic per-site defaults), DisplayValueArg. Flatten position is help order; a site whose help text differs keeps its own declaration
    init.rs         → sn init (onboarding wizard: profile setup + verification, and ALWAYS claims default_profile); a thin policy layer over profile.rs's core
    auth.rs         → session/identity core behind `sn profile login/logout/status/refresh` (no clap surface of its own; login runs the flow for a configured oauth profile, no config mutation) + whoami (authenticated-identity read, shared with profile.rs)
    profile.rs      → sn profile add/list/show/remove/use plus login/logout/status/refresh (dispatched to auth.rs) + the shared profile-writing core (ProfileInput, resolve_name/resolve_input, save_and_verify) used by both `add` and `init`
    table.rs        → sn table list/get/create/update/delete
    watch.rs        → sn watch <table> (live record watcher; streams JSONL)
    schema.rs       → sn schema tables/columns/choices (undocumented SN endpoints)
    api.rs          → sn api list/search/spec (instance API discovery over the REST API Explorer's undocumented doc endpoints)
    introspect.rs   → sn introspect (dumps clap command tree as JSON; see "The introspect contract")
    progress.rs     → sn progress + finish_cicd (poll async CICD ops)
    app.rs          → sn app install/publish/rollback (App Repository)
    update_set.rs   → sn updateset create/retrieve/preview/commit/commit-multiple/back-out
    atf.rs          → sn atf run/results (Automated Test Framework)
    aggregate.rs    → sn aggregate (server-side stats/counts/averages)
    scores.rs       → sn scores list/favorite/unfavorite (Performance Analytics)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tehubersheezy/servicenow-cli](https://github.com/tehubersheezy/servicenow-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
