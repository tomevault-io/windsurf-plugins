---
trigger: always_on
description: The operator + AI-driver guide for **droidsaw**: how to drive it and read what it tells you. It's
---

# Using droidsaw

The operator + AI-driver guide for **droidsaw**: how to drive it and read what it tells you. It's
written for the AI assistant driving the tool and for anyone who wants the lean operational reference.

The user-facing guide — install, [per-audience playbooks](README.md#playbooks),
[MCP wiring](README.md#mcp-server), the architecture, and the full [correctness story](README.md#correctness)
— lives in [`README.md`](README.md). This file is the operational layer on top of it.

> droidsaw is **static-only**. It reads an app as bytes, parses the container, decompiles the
> bytecode, and runs detectors entirely in memory. It never executes the sample and never makes a
> network request. That property is what makes it safe to point at hostile input.

---

## For AI assistants

If you are driving droidsaw:

- **Prefer the MCP tools** when the `droidsaw` server is active — it's active when `mcp__droidsaw__*`
  tools appear in your tool list. `load` once, then operate on the session. Otherwise use the CLI
  (`droidsaw <cmd> … | jq`).
- **Call `load` before any other MCP tool** — every other tool errors until the session is loaded
  (see the MCP section below).
- **Default to `audit --mode=basic`, explicitly** — fast, hermetic, no external tools. The binary
  default is `full`, which silently skips Semgrep/TruffleHog when they aren't installed; only reach
  for `full` when the user wants those and the binaries are present. (Via MCP the `audit` tool is
  classed `spawns-subprocess` for its worst-case mode, so even `basic` needs that class granted — that
  is a per-tool capability label, not a claim that basic spawns anything.)
- **`taint` and `query` are not CLI commands.** Taint comes from `audit`; query an exported DB with
  `sqlite3` (or the MCP `query` tool, which is `SELECT`-only).
- **stdout is always JSON.** Parse it; don't scrape human text. Errors are JSON envelopes with a
  non-zero exit code.
- **Resource caps exist** (`--budget-mem`, `--budget-time`) — use them on untrusted or large input in
  a constrained context.
- **Report honestly.** *Unrecognized* output and flagged-but-untraced flows are real signal, not
  failure — surface them as-is rather than papering over them.

---

## Build & run

```sh
cargo install droidsaw                 # the CLI
cargo install droidsaw --features mcp  # also installs the droidsaw-mcp server
cargo auditable install --locked droidsaw  # fleet/CI: binary embeds its dependency inventory
                                           # (verify later: cargo audit bin $(which droidsaw))
```

`cargo` pulls the `droidsaw-*` library crates from crates.io and compiles locally. Prereqs: a Rust
toolchain ([rustup](https://rustup.rs)) and a C compiler (`cc`/`clang`). `semgrep` and `trufflehog`
are optional (used by `audit --mode=full`, `--mode=semgrep`, `--mode=trufflehog`); YARA is built in. No
Java or Android SDK is needed. Install + build-from-source detail is in the [README](README.md#install).

**Building from a source checkout** (not needed to *use* droidsaw — `cargo install` above is the
normal path; PRs aren't currently solicited). The workspace wires its sibling library crates via a
`[patch.crates-io]` block that points at `../droidsaw-common`, `../droidsaw-cli-contract`,
`../droidsaw-dex`, `../droidsaw-hermes`, `../droidsaw-apk`. A bare clone of just this repo therefore
**fails** to build with a missing path-dependency error. Clone all six repos as siblings under one
parent directory, then build `droidsaw/`:

```sh
for r in droidsaw droidsaw-common droidsaw-cli-contract droidsaw-dex droidsaw-hermes droidsaw-apk; do
  git clone https://github.com/droidsaw/$r
done
cd droidsaw && cargo build   # the [patch.crates-io] block resolves the five siblings from disk
```

If you (an AI assistant) are asked to build a checkout and hit `error: failed to load source for
dependency ... ../droidsaw-<crate>`, the cause is missing sibling repos — clone them as above; do
not switch the user to `cargo install` (that installs the published crate, ignoring their checkout).

---

## I/O contract

- **Input** is an `.apk`, `.xapk`, `.hbc` (Hermes bundle), or `.dex`. DEX and Hermes layers are
  extracted from an APK automatically. Co-located `split_config.*.apk` siblings are auto-merged
  (`--no-auto-splits` to disable).
- **Output** is JSON on **stdout** — one object, one array, or an NDJSON stream. Nothing else.
  The documented plain-text exemptions: `--version` and `--help` (clap convention),
  `hbc disassemble`, `scan trufflehog`, and `decompile --all --js`.
- **Progress** goes to **stderr**, prefixed `droidsaw: `.
- **Exit code** is `0` on success, `2` on failure, and `1` is reserved for the opt-in
  `audit --fail-on=<critical|high|medium|low|info>` gate: the audit completed, stdout carries the
  normal audit output, and at least one emitted finding is at or above the threshold — gate a CI
  pipeline on the exit code alone. Every failure (including argv errors) is a typed JSON error
  envelope on stdout: `{ "error": { "code": "USER_INPUT|PERMISSION|TRANSIENT|CONFIGURATION|INTERNAL", ... } }`.
- **Output is deterministic** — same input, byte-identical output across runs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [droidsaw/droidsaw](https://github.com/droidsaw/droidsaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
