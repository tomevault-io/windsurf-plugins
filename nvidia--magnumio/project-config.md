---
trigger: always_on
description: provides an explicit RDMA address source.
---

# AGENTS.md

Guidance for AI coding agents, assistants, and copilots working on this
repository.

This project is a GPUDirect Storage (GDS) diagnostic toolkit. It helps users
decide whether a host is ready for GDS installation, whether GDS was installed
correctly, which filesystem/storage routes are supported, and what to fix when
GDS falls back to compat mode.

## Read These First

Use these project files as the local source of truth before changing behavior:

- `README.md` - user-facing commands and quick start.
- `doc/DESIGN.md` - command behavior, output contract, support-matrix semantics,
  and GDS-specific design rules.
- `skills/gds-diag/SKILL.md` - shared Claude/Codex diagnostic workflow for
  routing user intent to deterministic CLI commands and interpreting results.
- `doc/GDS_IMPROVEMENT_PLAN.md` - backlog and improvement rationale, when present.
- `checks/` and `subcommands/` - implementation.
- `tests/` - expected behavior and regression coverage.

For current NVIDIA behavior, also check the live host and current NVIDIA docs:

- GDS troubleshooting guide:
  https://docs.nvidia.com/gpudirect-storage/troubleshooting-guide/index.html
- CUDA downloads and installer flow:
  https://developer.nvidia.com/cuda-downloads
- CUDA Linux package-manager installation guide:
  https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html
- DOCA host/storage installation:
  https://docs.nvidia.com/doca/sdk/doca-host-installation-and-upgrade/index.html#storage-installation

Do not rely on memory alone for release-specific GDS support. If a claim depends
on current driver, CUDA, kernel, DOCA, MLNX_OFED, or filesystem release behavior,
verify against NVIDIA docs/release notes and local command output.

## Local Commands

Primary CLI:

```bash
python3 gds-diag.py --help
python3 gds-diag.py support-matrix
python3 gds-diag.py support-matrix --live
python3 gds-diag.py pre-install
python3 gds-diag.py post-install -v
python3 gds-diag.py mount-check /path/to/mount -v
python3 gds-diag.py config-audit --profile local-nvme
python3 gds-diag.py config-audit --config ./cufile.json --ignore-env -v
```

Development verification:

```bash
python3 -m unittest discover -s tests -v
python3 -m py_compile gds-diag.py checks/*.py subcommands/*.py tests/*.py
git diff --check
```

This project intentionally uses only the Python standard library for the main
CLI path. Do not add package dependencies unless the user explicitly accepts the
tradeoff and the dependency is isolated from basic command dispatch.

## Licensing

This directory is part of the Magnum IO repository and is distributed under
the repository's root Apache License, Version 2.0 (`../LICENSE`), with one
local exception for `skills/` (see below). Preserve the local notice files:

- `NOTICE`
- `LICENSE-CC-BY-4.0`
- `THIRD_PARTY_NOTICES.md`

Contribution and DCO sign-off guidance lives in the repository's root
`../CONTRIBUTING.md`; there is no separate `CONTRIBUTING.md` here.

For NVIDIA-authored source files, preserve or add concise SPDX headers using
file-appropriate comment syntax:

```text
SPDX-FileCopyrightText: Copyright (c) 2026, NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
```

Do not remove existing copyright, license, attribution, NOTICE, AUTHORS, or
third-party provenance information. If adding, vendoring, copying, or modifying
third-party code, document the source, license, and relevant notes in
`THIRD_PARTY_NOTICES.md` or the project's equivalent third-party notice file.

When auditing licensing, consider Git-tracked files in this repository. Do not
follow Git submodules, symlink targets outside the repository, or untracked
files unless the user explicitly expands the scope.

Preserve DCO sign-off guidance in the repository's root `CONTRIBUTING.md`. New
external contributions should use `Signed-off-by:` lines consistent with the
Developer Certificate of Origin.

## Command Boundaries

`pre-install` checks whether a host is ready before CUDA/GDS installation is
complete. It must not depend on GDS packages, `gdscheck`, `nvidia_fs`,
`/etc/cufile.json`, or `nvidia-smi`.

In `pre-install`:

- Use `lspci` for NVIDIA GPU presence.
- Treat missing CUDA Toolkit as FAIL.
- Treat missing NVIDIA Open Driver installation as FAIL.
- Print a `Runtime Versions` block, but do not call `nvidia-smi`; use
  `modinfo nvidia` for the driver module version. Report CUDA Toolkit,
  NVIDIA driver, nvidia-fs, and libcufile as version values only.
- Include a warning-only `DOCA / MLNX_OFED` advisory. Missing MLNX_OFED/DOCA is
  not a universal GDS blocker, but warn because RDMA paths and NVMe/NVMe-oF
  nvfs deployments that rely on NVIDIA storage-stack patches may need it.
- Keep driver version, compute capability, CoherentGPUMemoryMode, GPU BDFs, and
  GPU/NVMe topology out of pre-install. Those are post-install/runtime checks.
- Do not include an `nvidia-fs` section.
- Use the box-style renderer (`render_sections` from `checks/output.py`) for
  the main report. Keep long documentation URLs out of box cells; the final
  Mitigation Plan prints full URLs as standalone lines for terminals, SSH
  sessions, and logs.

`post-install` validates the installed runtime. It may use `nvidia-smi`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/MagnumIO](https://github.com/NVIDIA/MagnumIO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
