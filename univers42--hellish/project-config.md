---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`hellish` — a from-scratch, almost-POSIX shell in C (a 42 project grown well past the subject). It diffs byte-for-byte against `bash --posix`, runs under AddressSanitizer, stays 42-norm clean, and ships two compile-time-swappable allocators. The binary lands at `build/bin/hellish`.

Two git submodules are required: `vendor/libft` (stdlib + the `ft_malloc` allocator) and `vendor/scripts` (dev tooling). If a build fails on missing libft: `git submodule update --init --recursive`.

## Build matrix — OPT × SAFE

Everything goes through the root Makefile. Two independent knobs; the build banner announces the active allocator.

| Command | Optimization | Allocator | Sanitizers |
|---|---|---|---|
| `make` | `-O0 -g3` | libc (`SAFE=1`) | ASan + LeakSanitizer |
| `make OPT=1` | `-O3 -flto` | `ft_malloc` (`SAFE=0`) | none |
| `make SAFE=0` | `-O0 -g3` | `ft_malloc` | ASan (blind to ft_malloc) |
| `make OPT=1 SAFE=1` | `-O3 -flto` | libc | none |

- An explicit `SAFE=…` on the command line always wins over the per-mode default.
- libft builds into per-SAFE trees (`vendor/libft/build-libc` vs `build-ft`) and shell objects into per-mode trees (`build/obj` vs `build/obj-opt`), but the binary path is shared — `make test` / `make bench` force a relink so the binary always matches the requested mode.
- Flags are `-Wall -Wextra -Werror`: a warning is a build failure.

## Commands

```sh
make                # debug build (ASan+LSan, libc heap)
make OPT=1          # optimized build (ft_malloc heap)
make re             # fclean + rebuild, race-safe ordering (OPT/SAFE propagate)
make oracle         # build the PINNED bash 5.3.9 the suite is defined against
                    #   (cached in ~/bash-5.3.9; tests/tester auto-uses it)
make test           # full suite: ~3000 golden-diff cases vs bash --posix
make docker-suite   # same suite, hermetic: shell AND oracle from the image
make static         # static musl binary via docker -> dist/hellish-linux-<arch>
make static-verify  #   ... and prove it runs on THIS host
cd tests && ./tester redir pipe   # run only specific category files
cd tests && ./tester -v <file>    # verbose: show each case's diff
cd tests && ./verify_alloc.sh     # build BOTH heaps, prove output parity + no leaks
make bench          # speed vs bash --posix (always rebuilds OPT=1; ROUNDS=7 BENCH=micro)
make agnostic-bench # cross-shell speed matrix vs 8 shells, in docker (ROUNDS=… TIMEOUT_S=…)
make conformance    # third-party suites (Oils spec + mksh check.t) vs bash --posix AND dash
                    #   → bench/conformance.md; FAILS if pass count drops vs bench/baseline/
                    #   (UPDATE_BASELINE=1 to accept an improved count)
make perf           # dimension-split hyperfine bench (startup/parse/loops/forks/configure)
                    #   → bench/results.md; needs performance CPU governor, or BENCH_LAX=1
make geoman         # external 42 "minishell tester" as an independent cross-check (GEOMAN_URL=…)
make cli-opts-test  # shell's own argv parsing (-e, -o name, +c, --, $-) vs bash --posix
make login-test     # login shell sources /etc/profile then ~/.profile; non-login sources neither
make norm           # 42 norminette over src/ incs/ tests/ — REPORTS ONLY, always exits 0
make docker-test    # build + smoke-test from source on Alpine/Debian/Ubuntu/Arch
make docker-alpine  # (also -debian/-ubuntu/-arch) interactive hellish in that container
make cd-zsh-test    # docker diff of the zsh-style `cd old new` extension vs real zsh
make cd-posix-test  # host-side check that --posix gates that extension off
make hist-test      # pty-driven check of cmdhist multiline history joining
make readline-test  # pty gate over every libreadline entry point (completion,
                    #   history recall, vi/emacs) — run before touching readline linkage
make anim-test      # pty-driven check that the prompt animation never clobbers pasted input
make git-prompt-test # pty gate: the prompt's git dirty check never blocks a render — cd
                    #   into a slow-scanning repo must prompt instantly, star arrives async
make charts         # regenerate bench/charts/*.svg from whatever harness output is on disk
                    #   (never re-measures — charting and measuring stay separate)
make rss            # peak-RSS dimension alone (needs a prior `make perf` build)
make my_shell       # sudo-install to /usr/bin + register as a login shell (OPT=1 SAFE=1)
make user-install   # the SUDO-LESS route (user-install.sh): ~/.local/bin + a
                    #   marker-delimited `exec hellish` hook in the login shell's rc.
                    #   chsh cannot be used without root (shell must be in /etc/shells),
                    #   so the rc hook is the mechanism. Idempotent; smoke-tests the
                    #   binary BEFORE writing the hook. PREFIX=/RC_TARGET=/STATIC=1.
make user-uninstall #   ... strip the hook and remove the binary
```

Run it: `./build/bin/hellish [script.sh]`, `-c 'cmd'`, or pipe into it (non-TTY). Debug views compose: `--debug=lexer --debug=parser --debug=ast`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Univers42/hellish](https://github.com/Univers42/hellish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
