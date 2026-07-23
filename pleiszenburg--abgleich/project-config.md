---
trigger: always_on
description: > **Onboarding prompt** (paste at the start of a new session):
---

# AGENTS.md — abgleich project guide for LLM agents

> **Onboarding prompt** (paste at the start of a new session):
> "Read `AGENTS.md` in the project root thoroughly before doing anything else. It is the authoritative onboarding document for this codebase. Follow all conventions it describes."

---

## Project overview

**abgleich** is a ZFS management and synchronization CLI tool. It manages/backs up ZFS datasets across multiple machines, including over SSH with multi-hop routes. Written in Rust 2024 edition. The binary is called `abgleich`.

---

## Repository layout

```
abgleich-cli/           thin binary crate; main.rs → abgleich_lib::cli::entrypoint()
abgleich-lib/           library crate; all logic; feature-gated "cli" for clap/tracing-subscriber
docs/                   Sphinx docs; development.rst is the key dev guide
docs/top-level.just     doc build recipes (imported by root justfile)
tests/                  Python pytest suite
tests/lib/              test infrastructure (~31 modules)
tests/vagrant/          Vagrant + VirtualBox VM management (linux/, freebsd/, keys/)
tests/vagrant/top-level.just  VM control recipes (imported by root justfile)
justfile                root command runner; imports docs/ and tests/vagrant/ justfiles
bin/package.sh          packaging script
```

---

## Languages & tooling

| Tool | Purpose |
|---|---|
| Rust 2024 edition | core implementation |
| `just` ≥ 1.46 | command runner everywhere (host + VMs) |
| `cross` | cross-compilation (needs docker or podman) |
| `upx` | binary compression for Linux musl releases |
| `cargo-hack` | feature flag powerset checks |
| `clippy` pedantic+nursery | linting (aspired standard) |
| Python ≥ 3.12 | test suite |
| `pytest` | test runner |
| `vagrant` + `virtualbox` | test VM management |
| Sphinx | documentation |

Install Python test deps: `pip install -vr tests/requirements.txt`
Install doc deps: `pip install -vr docs/requirements.txt`

---

## Build system (`justfile`)

```bash
just build [TARGET]           # debug build via cross (default: x86_64-unknown-linux-musl)
just release [TARGET]         # release build + upx for Linux targets
just clippy                   # cargo clippy pedantic+nursery
just test-internal            # cargo test (internal Rust unit tests)
just test-features            # cargo-hack: all feature flag permutations
just fmt                      # cargo fmt
just clean-build              # remove build artifacts
just clean-dist               # remove dist/
just dist                     # package all targets
just docs-build [TARGET]      # build Sphinx docs (default: html)
just clean-docs               # remove built docs
```

Build targets: `x86_64-unknown-linux-musl` (default), `x86_64-unknown-linux-gnu`, `x86_64-unknown-freebsd`

**Local musl build without cross** (Linux):
```bash
rustup target add x86_64-unknown-linux-musl
sudo apt install musl-tools
```

**Cargo workspace** (`Cargo.toml`):
- Members: `abgleich-cli`, `abgleich-lib`
- Release profile: `lto=true`, `opt-level="z"`, `strip=true`, `panic="abort"`

**`abgleich-lib` features**: `cli = ["dep:clap", "dep:tracing-subscriber"]`

---

## Rust code structure (`abgleich-lib/src/`)

```
cli/            clap CLI; command.rs defines Commands enum; dispatch.rs routes commands
config/         YAML config detection + parsing; location.rs + route.rs
engine/         core ZFS logic (Engine, Apool, Dataset, Snapshot, comparison/, property/)
output/         table and storage display
subprocess/     Command, CommandChain, Outcome, Proc
transaction/    Transaction types, TransactionList, run_cli()
sys/            env, logging, errors
traits/         FromSerializable, ToSerializable, PathToString, Traverse
consts.rs       all constants and default values
```

### CLI subcommands (`cli/command.rs` → `Commands` enum)

| Subcommand | Flags | Args |
|---|---|---|
| `ls` | `-j/--json` | `[location]` (optional) |
| `snap` | `-j`, `-y/--yes`, `-f/--force` | `location` |
| `sync` | `-j`, `-y`, `-f`, `-d/--direct`, `-r/--rate-limit` (e.g. `10m`,`500k`), `-x/--compress` (level 0–9, default 5) | `source target` |
| `free` | `-j`, `-y`, `-f` | `source target` |
| `version` | — | — |

`--compress` without value defaults to level 5; omitting `--compress` entirely uses `zfs send -c`.

### Location format

```
[route:][user%]root
```
- `route`: optional SSH hop(s), e.g. `gateway/backupbox`
- `user`: optional ZFS command user; `root` → sudo
- `root`: root dataset; trailing `/` excludes root, includes descendants only
- Example: `gateway/backupbox:backupuser%tank/some/dataset`

### Config detection order

1. `ABGLEICH_CONFIG` env var
2. `./abgleich.yaml`
3. `~/.abgleich.yaml`
4. `/etc/abgleich.yaml`

### ZFS user properties (`abgleich:*`)

| Property | Type | Default | Meaning |
|---|---|---|---|
| `format` | string | `abgleich_%Y-%m-%dT%H:%M:%S:%3f_backup` | snapshot name format |
| `overlap` | int | `2` | snapshots to keep on source (-1 = all) |
| `diff` | bool | `true` | show diffs |
| `threshold` | u64 bytes | `12_582_912` | min data size to trigger sync |
| `snap` | enum | `changed` | `always` / `changed` / `never` |
| `sync` | bool | `true` | allow syncing |

### Subprocess abstraction

`Command` is immutable and chainable:
```rust
Command::new(program, args)?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pleiszenburg/abgleich](https://github.com/pleiszenburg/abgleich) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
