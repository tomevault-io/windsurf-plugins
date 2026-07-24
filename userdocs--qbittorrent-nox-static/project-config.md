---
trigger: always_on
description: Purpose: Make AI contributions precise, minimal, and correct. Follow these rules strictly. Do not expand scope beyond the prompt.
---

# AI Coding Guide for This Repo

Purpose: Make AI contributions precise, minimal, and correct. Follow these rules strictly. Do not expand scope beyond the prompt.

## Bash scripting (applies to all repos)

Do

- Use `#!/bin/bash` as the shebang for Bash scripts.
- Use the `.bash` extension for Bash; use `.sh` only for POSIX-only scripts.
- Prefer `$BASH_SOURCE` over `$0` for script path detection.
- Use `printf '%s'` for plain strings and `printf '%b'` for escape sequences. Avoid `echo`.
- Keep changes simple, modular, and scoped to the exact prompt.
- Write readable code; add concise comments explaining intent and non-obvious logic.
- Handle errors explicitly (per function is acceptable); return helpful, actionable messages.
- Structure changes in small stages; keep functions focused.
- Format using Google’s Shell Style Guide: https://google.github.io/styleguide/shellguide.html
- For Bash references, consult: https://mywiki.wooledge.org and https://mywiki.wooledge.org/BashFAQ and include a source link when possible. Do not invent links.

Avoid

- `set -euo pipefail`; prefer targeted checks and explicit error handling.
- Uppercase variable names for general scripting (reserve UPPERCASE for Docker/env settings).
- Clever one-liners that harm clarity.
- Generalized or speculative changes not asked for in the prompt.
- Over-engineering; keep it stable, concise, and C-like in mindset.

Scope and behavior

- Only implement what the prompt requests. All changes should be in the context of the prompt.
- Keep solutions minimal and modular; do not add placeholders or future-proofing unless required.
- When giving Bash/shell answers, add a relevant wooledge link if helpful; never fabricate links.

## GitHub Workflows (all repos)

- In reusable workflows, any job that uses outputs from another job must declare that job in `needs` to avoid null outputs.
- Do not use outdated Actions. Check for current recommended versions before editing.
- gh cli can get gets the id of a workflow it starts, here is an example usage

    ```bash
    run_url="$(~/bin/gh workflow run ci-main-reusable-caller.yml --repo user/repo -f skip_rerun=false -f retries=3)"
    ~/bin/gh run watch "${run_url##*/}" --repo user/repo --exit-status --compact --interval 30
    ```

## If repo name matches `*-musl-cross-make`

Toolchain specifics

- Use both `-static` and `--static` to produce static toolchain binaries. Using only `-static` can miss POSIX threads.
- When using `../config.mak`, always load options from both `../gcc-configure-options.md` and `../gcc-configure-options-recursive.md`.
- The binutils gold linker is deprecated. Use `ld=default` and `--disable-gold`.

Fully static toolchains with musl

- Do not use LTO: avoid `-flto` and `-fuse-linker-plugin`.
- Do not add any LTO-related settings.
- Only set linker options such as `LDFLAGS` at link time, not during library builds.
- GNU libtool redefines `-static`; to ensure static linking, use `--static` or `-Wl,-static` (optionally with `-static`) in `LDFLAGS`.
- For static OpenSSL: do not use `openssl -static` (it disables threads/PIE/PIC). For `-static-pie` with musl/Alpine, use the correct flags and approach.
- Do not use glibc-only flags or glibcisms for musl toolchains.

## Debugging with QEMU

- Start the target under QEMU with gdbstub, then attach with gdb:
    - `qemu -g <port> <binary>` (e.g., `qemu -g 1234 ./qbt-nox-static`)
    - In another terminal: `gdb ./qbt-nox-static` and `target remote :1234`

## If repo name matches `*qbittorrent-nox-static`

`qi.bash` script goals

- Simple installer that verifies installation and binaries.
- Shebang must be `#!/bin/bash`.

OS detection

- `source /etc/os-release`.
- Supported: `ID=alpine`, `ID=debian`, or `ID_LIKE` contains `debian`. Otherwise exit with a clear reason.

Transfer tools

- Prefer `curl` if present; use `wget` if present and `curl` is not; exit if neither is available.
- Detect presence of `gh` CLI and use it when available, but it is not required.

Architecture detection

- Alpine: `apk --print-arch`.
- Debian-like: `dpkg --print-architecture`.
- Architectures are the same across distros except `armhf`: Debian uses `armv7`, Alpine uses `armv6`.
- If architecture is not valid/supported, exit with a reason.

Download function

- Build the download URL from the detected architecture.
- Create and store the download’s SHA-256 sum.

Attestation (optional)

- When `gh` CLI is available and usable, verify downloaded binaries:
    - `gh attestation verify <INSTALL_PATH> --repo <REPO> 2> /dev/null`

Error handling

- Provide a helper that checks command exit codes and exits with a concise, helpful message on failure.

Output formatting

- Provide a print helper that supports:
    - `[INFO]` (blue), `[WARNING]` (yellow), `[ERROR]` (red), `[SUCCESS]` (green), `[FAILURE]` (magenta)
- Use `printf '%s'` and `printf '%b'`; do not use `echo`.
- Keep messages succinct. Be verbose only on errors to aid troubleshooting.

---

Meta for AI contributors

- Be conservative: do only what the prompt requests. No broad refactors.
- Prefer small, well-named functions and staged changes.
- Preserve existing public behavior and style unless the prompt requires changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [userdocs/qbittorrent-nox-static](https://github.com/userdocs/qbittorrent-nox-static) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
