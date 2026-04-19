---
trigger: always_on
description: `gws` is a Rust CLI tool for interacting with Google Workspace APIs. It dynamically generates its command surface at runtime by parsing Google Discovery Service JSON documents.
---

# AGENTS.md

## Project Overview

`gws` is a Rust CLI tool for interacting with Google Workspace APIs. It dynamically generates its command surface at runtime by parsing Google Discovery Service JSON documents.

> [!IMPORTANT]
> **Dynamic Discovery**: This project does NOT use generated Rust crates (e.g., `google-drive3`) for API interaction. Instead, it fetches the Discovery JSON at runtime and builds `clap` commands dynamically. When adding a new service, you only need to register it in `crates/google-workspace/src/services.rs` and verify the Discovery URL pattern in `crates/google-workspace/src/discovery.rs`. Do NOT add new crates to `Cargo.toml` for standard Google APIs.

> [!NOTE]
> **Package Manager**: Use `pnpm` instead of `npm` for Node.js package management in this repository.

## Build & Test

> [!IMPORTANT]
> **Test Coverage**: The `codecov/patch` check requires that new or modified lines are covered by tests. When adding code, extract testable helper functions rather than embedding logic in `main`/`run` where it's hard to unit-test. Run `cargo test` locally and verify new branches are exercised.

```bash
cargo build          # Build in dev mode
cargo clippy -- -D warnings  # Lint check
cargo test           # Run tests
```

## Changesets

Every PR must include a changeset file. Create one at `.changeset/<descriptive-name>.md`:

```markdown
---
"@googleworkspace/cli": patch
---

Brief description of the change
```

Use `patch` for fixes/chores, `minor` for new features, `major` for breaking changes. The CI policy check will fail without a changeset.

## Architecture

The CLI uses a **two-phase argument parsing** strategy:

1. Parse argv to extract the service name (e.g., `drive`)
2. Fetch the service's Discovery Document, build a dynamic `clap::Command` tree, then re-parse

### Workspace Layout

The repository is a Cargo workspace with two crates:

| Crate                          | Package                 | Purpose                                           |
| ------------------------------ | ----------------------- | ------------------------------------------------- |
| `crates/google-workspace/`     | `google-workspace`      | Publishable library — core types and helpers       |
| `crates/google-workspace-cli/` | `google-workspace-cli`  | Binary crate — the `gws` CLI                       |

#### Library (`crates/google-workspace/src/`)

| File             | Purpose                                                    |
| ---------------- | ---------------------------------------------------------- |
| `discovery.rs`   | Serde models for Discovery Document + async fetch/cache    |
| `services.rs`    | Service alias → Discovery API name/version mapping         |
| `error.rs`       | `GwsError` enum, exit codes, JSON serialization            |
| `validate.rs`    | Path/URL/resource validators, `encode_path_segment()`      |
| `client.rs`      | HTTP client with retry logic                               |

#### CLI (`crates/google-workspace-cli/src/`)

| File                | Purpose                                                                  |
| ------------------- | ------------------------------------------------------------------------ |
| `main.rs`           | Entrypoint, two-phase CLI parsing, method resolution                     |
| `auth.rs`           | OAuth2 token acquisition via env vars, encrypted credentials, or ADC     |
| `credential_store.rs` | AES-256-GCM encryption/decryption of credential files                  |
| `auth_commands.rs`  | `gws auth` subcommands: `login`, `logout`, `setup`, `status`, `export`   |
| `commands.rs`       | Recursive `clap::Command` builder from Discovery resources               |
| `executor.rs`       | HTTP request construction, response handling, schema validation          |
| `schema.rs`         | `gws schema` command — introspect API method schemas                     |
| `logging.rs`        | Opt-in structured logging (stderr + file) via `tracing`                  |
| `timezone.rs`       | Account timezone resolution: `--timezone` flag, Calendar Settings API    |

## Demo Videos

Demo recordings are generated with [VHS](https://github.com/charmbracelet/vhs) (`.tape` files).

```bash
vhs docs/demo.tape
```

### VHS quoting rules

- Use **double quotes** for simple strings: `Type "gws --help" Enter`
- Use **backtick quotes** when the typed text contains JSON with double quotes:
  ```
  Type `gws drive files list --params '{"pageSize":5}'` Enter
  ```
  `\"` escapes inside double-quoted `Type` strings are **not supported** by VHS and will cause parse errors.

### Scene art

ASCII art title cards live in `art/`. The `scripts/show-art.sh` helper clears the screen and cats the file. Portrait scenes use `scene*.txt`; landscape chapters use `long-*.txt`.

## Input Validation & URL Safety

> [!IMPORTANT]
> This CLI is frequently invoked by AI/LLM agents. Always assume inputs can be adversarial — validate paths against traversal (`../../.ssh`), restrict format strings to allowlists, reject control characters, and encode user values before embedding them in URLs.

> [!NOTE]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [googleworkspace/cli](https://github.com/googleworkspace/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
