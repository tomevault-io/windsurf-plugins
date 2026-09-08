---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# AGENTS.md

Guidance for AI agents working in this repository.

## What this repo is

The SaltStack state tree that builds **SIFT** (SANS Investigative Forensic Toolkit) — a
DFIR workstation image. It is **not** an application: there is no build step, no test
suite in the usual sense, and no code that runs on this machine. Every `.sls` file is a
Salt state that installs or configures forensics tooling on a target Ubuntu host.

Installation is driven by [cast](https://github.com/ekristen/cast), which reads
`.cast.yml`, downloads a single-binary Salt, and applies the state tree:

```console
sudo cast install teamdfir/sift-saltstack
```

Upstream repo: `git@github.com:teamdfir/sift-saltstack.git`. Issues live in the
[SIFT repo](https://github.com/sans-dfir/sift/issues) prefixed `[SALTSTACK]`.

## Supported targets

`.cast.yml` is the source of truth: **Ubuntu 22.04 (jammy)** and **24.04 (noble)**, on
**amd64 and arm64**. Salt **3006** and **3007** are both tested. Focal (20.04) is EOL and
should not be added to new code paths.

## Layout

```
.cast.yml                  cast manifest: modes, supported OS, pillar templates
VERSION                    vestigial (stale — see Known rough edges)
sift/
  desktop.sls  server.sls  the two real entrypoints
  vm.sls       pkgs.sls    legacy aliases -> desktop / server
  include-desktop.sls      desktop = server + config
  include-server.sls       server  = repos + python3-packages + packages + scripts
  repos/                   apt repos and PPAs (gift, sift, docker, microsoft, ubuntu-*)
  packages/                ~240 files, one apt package or .deb per file
  python3-packages/        tools installed into per-tool virtualenvs under /opt
  perl-packages/           CPAN modules via cpanm
  scripts/                 tools installed from tarballs/zips/git, plus wrappers
  config/                  hostname, timezone, folders, samba, user/ (desktop only)
  files/                   static payloads served via salt://sift/files/...
  tests/                   aggregate states used by the weekly CI job only
.ci/                       local dev helpers: test-state.sh, shell.sh
.github/workflows/         the CI that actually runs
```

Salt state IDs mirror paths: `sift/packages/foo.sls` → `sift.packages.foo`, and
`sift/packages/init.sls` → `sift.packages`.

## Entrypoints and the include graph

```
sift.desktop  -> sift.include-desktop -> sift.server -> sift.include-server -> repos
                                      -> sift.config                          python3-packages
                                                                              packages
                                                                              scripts
sift.server   -> sift.include-server
```

`cast` exposes these as modes: `desktop` (default) and `server`; `complete` and
`packages-only` are deprecated aliases.

## Conventions

Match the surrounding files. The tree is old and not uniformly consistent, so "what the
neighbours do" beats "what's globally most common."

### One tool per file

Each `.sls` under `packages/`, `python3-packages/`, `scripts/` installs exactly one tool.
Never bundle two tools into one file — CI tests changed states individually, so bundling
breaks the ability to test one thing.

### Register the new state in `init.sls`

Every directory's `init.sls` has a two-part structure that must be kept in sync:

```yaml
include:
  - sift.packages.newtool          # <- add here

sift-packages:
  test.nop:
    - name: sift-packages
    - require:
      - sls: sift.packages.newtool # <- and here
```

Adding to only one of the two lists is the most common mistake in this repo. A state
that is only in `include:` still runs, but the aggregate `test.nop` no longer gates on
it, so ordering guarantees quietly weaken.

Exception: a package that exists purely as a dependency of another state does not need
an `init.sls` entry — it gets pulled in transitively by whoever `include:`s it. 24 files
under `packages/` are dependency-only this way (`libicu`, `zlib1g-dev`,
`software-properties-common`, `dotnet`, …). Only add to `init.sls` when the tool should be
installed on its own merit.

But a state in neither `init.sls` **nor** any other state's `include:` is dead — it never
runs, and nothing reports that. Fourteen packages sat that way for years, several of them
receiving "updated for 24.04" commits while unreachable. Before adding a dependency-only
state, confirm something actually includes it.

### Header comment block

Tool-facing states carry a metadata header (present on ~66 of 240 package files, and on
most newer additions). Include it on anything user-visible; skip it for pure build deps.

```yaml
# Name: AESKeyFinder
# Website: https://citp.princeton.edu/our-work/memory/
# Description: Find 128-bit and 256-bit AES keys in a memory image.
# Category:
# Author: Nadia Heninger, Alex Halderman
# License: Free, unknown license
# Notes: aeskeyfind
```

`Notes:` is the list of binaries the tool puts on `$PATH` — keep it accurate, it is the
only machine-readable record of what SIFT provides.

### State ID naming

Two conventions coexist in `packages/`: bare `<pkgname>:` (128 files, the older style)
and `sift-package-<pkgname>:` (91 files, the newer style). **Use the `sift-` prefix for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teamdfir/sift-saltstack](https://github.com/teamdfir/sift-saltstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
