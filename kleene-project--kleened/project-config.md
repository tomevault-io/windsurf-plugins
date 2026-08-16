---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Kleened is the backend daemon of **Kleene**, a container management tool for FreeBSD built on
jails, ZFS and pf. It is an Elixir/OTP application exposing a REST + WebSocket API; the
reference client is the `klee` Python CLI, whose HTTP client is generated from this repo's
OpenAPI spec.

When working inside the `kleene_dev` checkout, `../CLAUDE.md` documents the QEMU/libvirt
development VM, the `klee` CLI and the docs site. This file covers the daemon itself.

When implementing new features, remember to adjust the docs site and relevant docs in
the klee and kleened repositories.

## Runtime constraint — read first

Kleened only runs on **FreeBSD**, as **root**. It shells out to `/usr/sbin/jail`, `zfs`,
`pfctl`, `ifconfig` and `mount_nullfs`, so there is no meaningful way to exercise it on Linux
or macOS.

- Runs anywhere: `mix compile`, `mix format`, `mix dialyzer`. The dialyzer workflow
  (`.github/workflows/dialyzer.yml`) deliberately runs on `ubuntu-latest` for this reason.
- Requires a FreeBSD host as root: `mix test`, and anything touching `Core.ZFS`,
  `Core.FreeBSD`, `Core.Network`, `Core.Exec` or `Core.Mount`.

The test suite assumes a prepared host:

- `/usr/local/etc/kleened/config.yaml` (copy `example/kleened_config_dev.yaml`)
- `/usr/local/etc/kleened/pf.conf.kleene` (copy `example/pf.conf.dev.kleene`)
- `/usr/local/etc/kleened/certs/` (copy `test/data/test_certs/`)
- a `zroot/kleene_basejail` dataset holding an extracted FreeBSD `base.txz`
- `mix run --eval "Kleened.Core.Config.initialize_host(%{dry_run: false})"`, run once
- `kldload if_bridge`, `service pf start`, `service pflog start`,
  `sysctl net.link.bridge.pfil_bridge=1` — without the last one some networking tests fail

`.github/workflows/run_tests.yml` is the executable version of that list; consult it rather
than reinventing the setup.

## Commands

All targets live in `Makefile` and must be run as root on the FreeBSD host.

```sh
make test          # mix test --seed 0 --trace --max-failures 1
make shell         # MIX_ENV=test iex -S mix
make test-shell    # same, but creates the FreeBSD:testing base image first
make codecov       # excoveralls HTML report into ./coveralls/
make release       # mix release --overwrite
make init          # Config.initialize_host, for real
make dryinit       # Config.initialize_host, dry run
make runpty        # compile c_src/runpty.c -> priv/bin/kleened_pty
```

Running a single test file or line requires `priv/bin` on `PATH` (see below):

```sh
PATH=$PATH:./priv/bin mix test test/network_test.exs
PATH=$PATH:./priv/bin mix test test/network_test.exs:412
```

**`priv/bin` must be on `PATH` for any test that allocates a TTY.** `Core.OS.cmd_async/2`
locates the `kleened_pty` helper with `:os.find_executable`, not by path. The Makefile's
`PTY_PATH` variable exists for this; note its comment explaining why it is *not* named `PATH`
(GNU make would export it and clobber the real one, so `mix` would no longer be found).

Static analysis:

```sh
make dialyzer-plt  # slow (minutes); only needed once, and again when mix.lock changes
make dialyzer
mix format         # .formatter.exs covers {mix,.formatter}.exs and config/ lib/ test/
```

Regenerating the API spec:

```sh
mix openapi.spec.json --spec Kleened.API.Spec   # writes ./openapi.json (gitignored)
```

## Architecture

### Supervision tree

`lib/core/application.ex` starts, in order: `Core.Config`, `Core.MetaData`, `Core.Network`, a
`Registry` named `Core.ExecInstances`, a `DynamicSupervisor` named `Core.ExecPool`, then one
`Plug.Cowboy` child per configured listening socket. The pool runs with `max_restarts: 0` — a
crashed exec instance is not retried. Once the tree is up, containers whose `restart_policy`
is `"on-startup"` are started.

### Config

`lib/core/config.ex` has two distinct entry points:

- `bootstrap/0` runs *before* the supervision tree and returns the cowboy listener specs
  parsed from `api_listening_sockets`.
- `initialize_host/1` is a separate one-shot host-preparation path (load `zfs`/`pf`/`pflog`
  kmods, set the corresponding `sysrc` entries, create the root dataset, enable `rctl`). It is
  what `make init` and the `kleened init` rc-script command call.

At runtime config is a string-keyed map behind an `Agent` (`Config.get/2`). Several keys are
derived in `initialize/0` and never appear in the YAML: `container_root`, `image_root`,
`volume_root`, `metadata_db`, and `host_gateway` (detected from the routing table).

### MetaData

`lib/core/metadata.ex` is an `Agent` wrapping a SQLite connection (exqlite). Records are
stored as **JSON blobs** in `networks`, `images`, `containers`, `volumes`, `mounts` and
`endpoint_configs`, and queried with SQLite's `json_extract`/`json_insert`. The
`api_list_containers` view joins image name/tag onto each container row.

Rows are decoded back into `Schemas.*` structs by `transform_row/2`, which dispatches on the
query's **column names**. Changing a `SELECT` list therefore silently changes which struct
comes back — keep column names in sync with that function.

### API layer


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kleene-project/kleened](https://github.com/kleene-project/kleened) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
