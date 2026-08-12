---
trigger: always_on
description: A single static Go binary that reports Claude, Codex, and Copilot usage limits from the terminal — and switches the live credential between stored Claude or Codex accounts without a browser round-trip. JSON by default; `-h`/`--human` for the nested text rendering.
---

# aistat

A single static Go binary that reports Claude, Codex, and Copilot usage limits from the terminal — and switches the live credential between stored Claude or Codex accounts without a browser round-trip. JSON by default; `-h`/`--human` for the nested text rendering.

CLI surface (provider-scoped with bulk-on-omission as of the multi-account effort):

```
aistat                                 # default: same as `aistat usage`
aistat usage [provider]                # report usage across all providers/accounts
aistat usage --refresh                 # bypass 90 s per-account cache; writes through

aistat switch                          # fan out to every provider with ≥ 2 stored accounts (auto-pick best by headroom)
aistat switch <provider>               # target one provider; auto-pick
aistat switch <provider> --to <id>     # target one provider, one stored account (email or uuid)
aistat switch --to <id>                # infer provider from id-uniqueness (no provider arg)
aistat switch <provider> --if-above-5h <N|off>       # conditional: only switch when the active account crosses a threshold, else "no switch needed"
aistat switch <provider> --if-above-weekly <N|off>   # presence of ANY threshold flag ⟹ conditional
aistat switch <provider> --if-above-5h 90 --notify   # + desktop notification on switch or on a hit with no better account (macOS)
                                        # threshold precedence per window: flag > env AISTAT_IF_ABOVE_5H / AISTAT_IF_ABOVE_WEEKLY > default 85/95; "off" disables a window

aistat switch <provider> --watch       # run the conditional switch on a timer in the foreground (implies conditional; always notifies, dedup'd across ticks); daemonize via launchd/systemd
                                        # --watch/-w + --interval (default 300s, min 60); with no threshold flag the windows fall back to env/default

aistat accounts list                   # list every provider's stored accounts (JSON: {claude:[...], codex:[...]}; text -h: section headers)
aistat accounts list <provider>        # scope to one provider
aistat accounts remove <id>            # infer provider from id-uniqueness; error if id matches > 1 provider
aistat accounts remove <id> <provider> # explicit provider

aistat -h | --human                    # text rendering (affects `usage` and `accounts list`)
aistat --debug                         # per-request / per-provider lines on stderr
aistat --version
aistat --help
```

JSON is the default; exit codes: 0 success / 1 any-provider-failed / 2 usage error / 3 stdout-write error.

`aistat switch --watch` is a foreground loop kept alive by the OS service manager (launchd `KeepAlive` / systemd `Restart=always`) — there is deliberately no `install`/`uninstall`/`status` subcommand; the unit file/plist is the "install".

## Repo layout

- [cmd/aistat/](cmd/aistat/) — `main`, hand-rolled subcommand dispatch (`scanGlobals` + per-subcommand FlagSets), provider registry, fake-provider hooks. One file per subcommand: [main.go](cmd/aistat/main.go), [usage.go](cmd/aistat/usage.go), [switch.go](cmd/aistat/switch.go) (CLI-private `switchable` interface + `buildSwitchHandles` registry + bulk / single / inferred dispatch, threshold-flag parsing via `resolveThreshold`, the `--watch` branch + `routeConditional`), [switch_conditional.go](cmd/aistat/switch_conditional.go) (`switchOpts` + `triggerReason` / `usageSummary` + `notifyBestEffort`), [watch.go](cmd/aistat/watch.go) (loop primitives for `switch --watch`: `newDedupNotifier` + `watchLoop` + `sleepWithCtx` + `watchThresholdDisplay`; the entry point is `runSwitch`'s `--watch` branch), [accounts.go](cmd/aistat/accounts.go) (`providerStore` + multi-provider list/remove with id-uniqueness inference).
- [internal/accounts/](internal/accounts/) — provider-neutral persisted account store. `Account` is opaque identity + metadata + `RawBlob`; provider packages own credential-shape parsing. `Provider` (closed set: `ProviderClaude`, `ProviderCodex`) with `validate()` gating all `OpenStore` calls. Backends: `store_darwin.go` keychain at `aistat:accounts:<provider>:<uuid>` + process flock on `$CACHE/aistat/store.lock`; `store_file.go` (`linux || windows`) JSON at `~/.config/aistat/accounts/<provider>.json` (`%USERPROFILE%\.config\...` on Windows) locking a `.<provider>.lock` sentinel (NOT the data file — survives atomic-rename) via `syscall.Flock` (`store_file_unix.go`) or `winlock.Lock` (`store_file_windows.go`); `MemoryStore` for tests. The shared body exposes a `withLock(exclusive bool, …)` seam so the lock primitive is the only per-OS code.
- [internal/providers/](internal/providers/) — one subpackage per provider (`claude`, `codex`, `copilot`). Each owns its credential source, HTTP calls, and response normalization into the shared `Limit` type in [types.go](internal/providers/types.go). `AccountResult` lives here too — same type carried on both `ProviderOutput` (in-process) and `ProviderResult` (JSON-serialized).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drogers0/aistat](https://github.com/drogers0/aistat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
