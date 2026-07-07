---
trigger: always_on
description: A Suckless-style PTY proxy in Zig 0.16. Sits between a terminal emulator and a shell; composes its middleware (atuin autosuggest, guardrail, history) **at compile time** via an `inline for` over a config tuple. No plugin loader, no `*anyopaque`, no runtime branching on the module list.
---

# atty — agent orientation

A Suckless-style PTY proxy in Zig 0.16. Sits between a terminal emulator and a shell; composes its middleware (atuin autosuggest, guardrail, history) **at compile time** via an `inline for` over a config tuple. No plugin loader, no `*anyopaque`, no runtime branching on the module list.

User is on omarchy/Hyprland with Ghostty. Project hosted at `github.com/fentas/atty`, master is the main branch, release-please-driven CI.

## Build & test

```sh
zig build -Dtarget=x86_64-linux-gnu -Doptimize=ReleaseSafe   # build atty
zig build test  -Dtarget=x86_64-linux-gnu                    # unit tests
zig build itest -Dtarget=x86_64-linux-gnu                    # PTY integration tests
zig build e2e   -Dtarget=x86_64-linux-gnu                    # scripted-PTY scenarios + visual diff
zig fmt --check src/ build.zig
```

`-Dtarget=x86_64-linux-gnu` is **required on this dev box** — Arch's gcc-16 crt1.o has SFrame relocations Zig 0.16's linker can't handle. Without the flag the build fails with `R_X86_64_PC64`. CI uses musl targets so doesn't hit this.

`make build` / `make test` / `make itest` / `make e2e` wrap the above. `make link` symlinks `~/.local/bin/atty` to `./zig-out/bin/atty` for dev.

## File layout (most-touched)

```
src/
├── main.zig              CLI entry (atty [shell [args]])
├── root.zig              library entry — re-exports for @import("atty")
├── proxy.zig             poll() loop, signals, ghost-text, statusbar
├── module.zig            shared types: Action, Context (incl. ctx.incognito), Error
├── dispatch.zig          Dispatcher(modules) — inline-for walker; MouseAction
├── pty.zig               posix_openpt / grantpt / fork+exec child
├── mouse.zig             SGR-1006 mouse-event parser (DECSET ?1000h?1002h?1006h)
├── line_state.zig        best-effort user-input buffer model + uncertain flag
├── ghost.zig             ghost-overlay state machine
├── statusbar.zig         DECSTBM bottom row
├── ansi.zig              minimal SGR/CSI helpers
├── style.zig             atty.Style + atty.style.presets
├── keymap.zig            Action + Binding + key("Ctrl+Shift+I")
├── defaults.zig          atty-shipped defaults — single source of truth
├── config_resolver.zig   merges user_config + defaults via @hasDecl
├── config.def.zig        committed template (atty maintains)
├── config.zig            user's overrides — GITIGNORED, seeded by build.zig
├── modules/
│   ├── _lib.zig          shared helpers for built-in modules (nowMs, ListBuilder)
│   ├── atuin.zig         async worker; ghost + record + sync + pick list
│   ├── guardrail.zig     dangerous-command confirmation
│   ├── history.zig       shell-native ~/.bash_history fallback + pick list
│   ├── mouse_links/      left-click on a path token → `$EDITOR +LINE 'path'\n`
│   ├── mouse_urls/       left-click on a URL → xdg-open, gated by trust posture
│   └── security_guard/   pre-Enter Tier-1 + UDS client to atty-guard sidecar
│                         + Alt+Shift+W scrollback dump of buffered warn events
└── test/
    ├── integration.zig   real-PTY tests (zig build itest)
    └── e2e/              .e2e DSL scenarios + VT-grid diff harness

atty-guard/                  Rust sidecar daemon — system service running
│                            as `atty:atty` user/group (post-#140). UDS
│                            server with two-tier classifier (Tier-1
│                            regex/atom + Tier-2 SLM/heuristic), V2-J
│                            multi-hit accumulator, V2-J-2 opt-in auto-
│                            Block (`[accumulator] block_threshold`; red
│                            `REFUSED` line atty-side), eBPF LSM +
│                            execve/AF_ALG tracepoints, OSV live npm
│                            lookup, atom fetcher (GTFOBins + sanitized
│                            Sigma; LOLBAS dropped — Windows-native, see
│                            git log on atom_fetcher.rs for rationale).
│                            Mediated CLI: `sudo atty-guard atoms/urls/
│                            session/trust …` for per-UID mutations
│                            (SO_PEERCRED-gated). Three-source atom
│                            overlay scanned per classify: bundled
│                            (include_str!) → /var/lib/atty-guard/
│                            atoms.system.txt (daemon-fetched, perm-
│                            gated atty:atty 0640) → /var/lib/atty-guard/
│                            users/<uid>/atoms.user.txt (sudo-mediated).
│                            Install: `sudo make install-guard
│                            [GUARD_FEATURES=...,ebpf]` — the ebpf flag
│                            triggers the systemd drop-in auto-install.
│                            Opt-in commit pinning for the atom corpus
│                            via /etc/atty-guard/atoms.pins.toml
│                            (root:root, deny_unknown_fields, hard fail
│                            on typos; example template ships next to
│                            it; drop the file to opt out).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fentas/atty](https://github.com/fentas/atty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
