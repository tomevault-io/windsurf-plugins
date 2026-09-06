---
trigger: always_on
description: Kerness is a framework for building **multi-agent harnesses**: sessions in which
---

# Kerness

## Mission

Kerness is a framework for building **multi-agent harnesses**: sessions in which
several language models hold a structured conversation, call tools, consult a
shared memory, and produce a result with named fields.

The organising idea is that a **Markdown gameplan is the program**. Its YAML
frontmatter is a machine-readable contract — who the agents are, how many
rounds, which phases, which tools, what the result must contain — and its body
is the prose manual the orchestrator reads. A harness author writes Markdown;
the framework validates it, assembles the prompts, drives the loop, dispatches
the tools, and enforces the access boundary.

Two properties follow from that and shape every decision below:

- **The contract is total.** Every key the frontmatter parser accepts is
  validated, rendered into a prompt, or enforced at runtime. A key that parses
  and then does nothing is a bug, not a reserved word.
- **Everything is synchronous.** There is no executor, no async runtime, and no
  hidden concurrency. A session runs on the calling thread, and a stack trace
  from inside a tool handler reaches the calling `SessionRun::step` or `Session::run`.

Kerness ships as two artifacts from one repository: a **Rust crate** for callers
who want the framework in a Rust program, and a **Python extension** for callers
who want to subclass `Provider`, pass a lambda as a tool handler, and hand a
`pydantic` model in for structured output. Both are first-class surfaces;
neither is a wrapper around the other's use case.

What is not symmetrical is where the code lives. **A feature is written in
Rust.** The crate implements it, the extension exposes it, and the installed
Python package does one of five things and nothing else: declares a class
callers subclass (`Provider`, `Channel`, `MemoryStore`), declares one the
extension cannot
(the exception hierarchy's structured constructors, `ToolDialect` as a real
`enum.Enum`, `AccessPolicy` as a dataclass whose contract is written in Python
list semantics), reads a signature with `inspect`, validates with `pydantic`,
or re-exports a name. Every other `.py` in the package is a shim.

Where a feature needs something only the interpreter has — `sys.stdout`, a
logger, `input`, an HTTP client under a caller's `mock.patch` — the crate names
the need as a trait, ships a default that works from Rust alone, and the
binding installs a replacement at `bootstrap`. The behaviour stays in one place
and only its delivery crosses:

| Seam | Crate default | What the binding installs |
| --- | --- | --- |
| `HttpTransport` (`http.rs:24`) | `ureq` + `rustls` | `kerness.provider.http_post_json`, resolved per call so `mock.patch` reaches it |
| `Logger` (`logging.rs:28`) | warnings and errors to stderr | `logging.getLogger("kerness")`, so `caplog` sees them |
| `ConsoleWriter` (`channel.rs:54`) | this process's stdout | `builtins.print`, so `capsys` and a `StringIO` see it |
| `ConsolePrompt` (`access.rs:69`) | `std::io::stdin` | `sys.stdin` / `builtins.input` |

## Target Environment

| | |
| --- | --- |
| Rust | MSRV **1.88**; stable toolchain; 2021 edition |
| Python | **3.10+**, CPython, via the stable ABI (`abi3-py310`) |
| Bindings | `pyo3` 0.23 with `extension-module` |
| Build | `cargo` for the crate, `maturin` for the wheel |
| Platform | Linux and macOS; developed on Linux x86-64. `ureq` + `rustls` reach further, but path confinement resolves every path from `/` — `crates/kerness/src/access.rs:713` — so the access boundary assumes POSIX paths |
| Network | Outbound HTTPS only, to provider endpoints the caller names |
| Runtime deps | None beyond the crate's Cargo dependencies; `pydantic` is optional and only for structured output |

There is no daemon, no database, no listening socket, and no background thread.
Filesystem writes are confined to paths the caller opts into: whatever the
memory store names for a scope, the session file, channel logs, and directories
added to the access policy.

A built wheel is tagged `kerness-<version>-cp310-abi3-<platform>` — one wheel
per platform covers every supported Python, and `<version>` is whatever the root
`Cargo.toml` says.

## Workspace Layout

One artifact per top-level directory: `crates/` is the crate, `bindings/` is
everything the wheel is made of. Neither reaches into the other's tests, and the
root carries one manifest — `Cargo.toml`. A Python build starts from
`bindings/python/`, not from here.

```
Cargo.toml                  workspace root, shared dependency versions
crates/
  kerness/                  the framework — pure Rust, links no Python
    src/                    31 top-level modules plus provider/, skill/, and session/,
                            documented by subsystem in the Index
    assets/                 built-in gameplans, roles, personas, skills
    tests/                  integration tests over the crate's public surface
    examples/               10 harnesses driven from Rust alone
bindings/
  python/                   everything the wheel is built from
    pyproject.toml          the wheel's manifest; `pip install .` runs here
    Cargo.toml              the `kerness-py` crate, a workspace member
    LICENSE  README.md      symlinks to the root copies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xwings/kerness](https://github.com/xwings/kerness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
