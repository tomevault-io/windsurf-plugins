---
trigger: always_on
description: validates them. There is no jailer configuration or direct-launch fallback. Existing direct
---

## Project

nibrunner is one binary, `nibrunnerd`, that turns a Linux machine with `/dev/kvm` into a microVM
host. A JSON document names the apps; the daemon boots each one in its own Firecracker microVM,
routes its hostname, sleeps and wakes it, and keeps its volume.

Rust workspace (`crates/*`) with the docs site under `docs/`.

## Stack

- **Language:** Rust; the host binary is one static x86_64 musl binary
- **Tasks:** `just`; `mise` installs the tools pinned in `mise.toml`
- **Linter/Formatter:** rustfmt and clippy for the Rust, Biome (`docs/biome.json`) for the docs
  site; editors format on save
- **State:** SQLite through sqlx, with the offline query data committed under `.sqlx/`. New
  tables store fields in columns and related rows, without JSON records.
- **Docs:** Fumadocs on Bun under `docs/`, served at [nibrunner.dev](https://nibrunner.dev)
- **Commits:** Conventional Commits, checked on pull request titles by `check-pr-title`

## Layout

- `crates/protocol` — the control-plane protocol: the desired and reported state documents.
  `schema/*.json` is generated from its types
- `crates/oci-image` — validates OCI image archives and assembles their filesystem layers without
  requiring Docker or a registry client
- `crates/guest-contract` — what host and guest agree on: drive order, kernel args, vsock ports,
  the frame codecs
- `crates/nft-render` — slot arithmetic, the nftables ruleset rendered whole, the parsers for what
  `nft` answers
- `crates/nibrunnerd` — the daemon. `domain/` holds the logic and needs no kernel; `ports.rs`
  names what it needs from outside as traits; `adapters/` implement them against Firecracker,
  nft, taps, volumes and the proxy; `repositories/` are the SQLite tables; `services/` wire
  domain to adapters and `controllers/` are the passes that run on a clock; `install/` lays a
  host out; `config.rs` is `config.toml`
- `crates/init` — the guest's PID 1
- `crates/test-tenant` — the tenant the guest tests boot: one static binary that answers, writes,
  remembers and misbehaves on demand
- `guest/` — the kernel, the image build script and the manifest of pins and digests
- `deploy/` — the installer, the systemd unit, and the generated `config.example.toml` and
  `config.schema.json`
- `docs/` — the site; pages are MDX under `docs/content/docs/`

## Code style

- **Code must be self-explanatory — this is strict.** Express intent through naming, types and
  structure, not prose. Do not write a comment that restates what the code already says. A comment
  is warranted only for what the code cannot say: a tradeoff, an external constraint (a
  Firecracker, nftables, kernel or ext4 quirk; a contract with the control plane), a maintenance
  note. Why, never what. If you feel the need to explain _what_ a block does, rename or extract it
  instead. Delete comments that no longer earn their place. This holds in the justfile and the
  workflows too: a step's `name:` is its label, and a comment on it is only for a why the command
  hides.
- The workspace lints are in `Cargo.toml`, and `just lint` runs clippy with `-D warnings` for
  default-feature libraries and binaries, then all targets with all features. Keep both passes:
  test dependencies enable `schema` and `testing`, which can hide warnings in production builds.
  `unwrap_used` and `panic` are among the lints: production code returns an error, or
  `expect`s with the reason it cannot fail. Tests and the `testing` feature are exempt, in `lib.rs`.
- `unsafe_code` warns too. An `unsafe` block sits in the smallest item that can hold it, under
  `#[allow(unsafe_code)]`, with a `reason` when it is not obvious.
- `unreachable_pub` warns: `pub` only what another crate reaches, `pub(crate)` otherwise.
- rustfmt: `max_width = 110`, imports and modules reordered. Do not fight it.
- A proper noun in a doc comment that clippy takes for an identifier goes in `clippy.toml`'s
  `doc-valid-idents`, not in backticks.
- Errors are `thiserror` enums whose messages read as the sentence an operator will find in a log.
- The daemon's CLI uses clap derives in `crates/nibrunnerd/src/cli.rs`. Declare commands, flags
  and their help there; clap handles parsing, help and argument errors.
- A metric is a `Metric` static beside the pass that emits it — name, help, kind and labels in one
  place, because `# HELP` and `# TYPE` are optional and a page whose two halves name a series
  differently scrapes without complaint. Its prefix says what is measured: `nibrunner_app_` for one
  app, `host_` for the machine, `volume_`/`checkpoint_`/`export_` for what it holds, `proxy_` for
  the listener, `vm_` for an operation on a microVM. `metrics::declared()` is the catalogue, and a
  test holds the page to emitting every series in it.
- Tests are named as the sentence they prove: `a_booted_vm_is_not_a_running_app`, not `test_boot`.
- Cron policy belongs to the host's `[cron]`: the job limit for each app and the IANA time
  zone, parsed with `chrono-tz`. Both settings are required, without defaults. Jobs are registered
  through `crontab`; desired state does not carry cron configuration.
- Cron runs may overlap. Editing a crontab reschedules future runs; stopping or replacing its

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ilbertt/nibrunner](https://github.com/ilbertt/nibrunner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
