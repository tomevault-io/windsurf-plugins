---
trigger: always_on
description: Purpose: Guide AI changes to the nrsyncd OpenWrt 802.11k Neighbor Report synchronization daemon.
---

# Copilot Instructions (nrsyncd)

Purpose: Guide AI changes to the nrsyncd OpenWrt 802.11k Neighbor Report synchronization daemon.

## Components

- `service/nrsyncd.init`: procd init; gathers per‑iface NR JSON, builds `SSIDn=` TXT args + metadata (`v=1 c=<count> h=<hash>`), exports env, registers mDNS service, admin subcommands.
- `bin/nrsyncd`: Prebuilt daemon (opaque) consuming ordered `SSIDn=` args; periodic neighbor aggregation, mDNS refresh, metrics/state updates; signals HUP/USR1/USR2.
- `lib/nrsyncd_common.sh`: POSIX helpers (adaptive retry, iface mapping, ms sleep abstraction, readiness probes, normalization).
- `scripts/install.sh`: Idempotent installer (deps, sysupgrade persistence, wireless validation / optional auto‑fix, prefix staging, opkg/apk detection, migration helpers).
- `package/nrsyncd/`: OpenWrt package skeleton (Makefile + default `/etc/config/nrsyncd`).
- `tests/` harness (mocks + scenarios: basic, skip, reload; installer test).
- `examples/wireless.config`: Example dual‑band wireless config with required 802.11k / BSS transition options.

## Runtime Flow

1. Count enabled `wifi-iface` stanzas.
2. Wait for matching `hostapd.*` ubus objects (3s cadence, 5m timeout).
3. Quick per‑iface retrieval via `rrm_nr_get_own` (upstream hostapd ubus method) with adaptive retry.
4. Build `SSIDn=` tokens (ordering invariant) then append metadata tokens.
5. Launch daemon with args; export `NRSYNCD_*` env (legacy `RRM_NR_*` exported during deprecation window if still needed).
6. Advertise `_nrsyncd_v1._udp` (browse may still observe `_rrm_nr._udp` for interoperability).
7. Daemon cycles: compute neighbor reports, refresh mDNS (rate‑limited), update metrics/state; honor signals.

## Conventions

- Delimiter `+` reserved; never alter SSID content.
- Never renumber existing `SSIDn=` positions; only append new higher `n` or metadata tokens at end.
- Metadata keys present: `v` (schema version), `c` (SSID entry count), `h` (first 8 hex of md5 over concatenated SSID tokens). Future keys must append only.
- Error hard‑fail for structural config issues; bounded adaptive retry for readiness (no hostapd restarts).
- Shell: strict POSIX (BusyBox ash); avoid arrays, bashisms.
- Skip list via `list skip_iface <iface>`; normalization strips optional `hostapd.` prefix.
- Metrics/state files: `/tmp/nrsyncd_runtime`, `/tmp/nrsyncd_metrics`; ephemeral.
- Version constant: `NRSYNCD_INIT_VERSION` in init script (bump each tagged release).

## Safe Extension Guidelines

- Add new TXT metadata only after all `SSIDn=` tokens; ensure daemon ignores unknown tokens.
- New UCI options: document (README + this file) & ensure reload semantics (SIGHUP path) handle them.
- Keep init fast; heavy logic belongs in daemon or future sidecar.
- Limit retries for non‑transient failures (≤3) with single concise log line per iface.
- If adding per‑iface disable: `option nrsyncd_disable '1'` and mirror skip logic in counting/enumeration.

## Developer Workflow

- Install: `sh scripts/install.sh` (flags: `--fix-wireless`, `--add-sysupgrade`, `--deps-auto-yes`, `--prefix <dir>` etc.).
- Manual deploy: copy binary/init/lib; enable & start service.
- Package build: place `package/nrsyncd` into OpenWrt buildroot `package/`; select in `menuconfig`.
- Debug startup: `sh -x /etc/init.d/nrsyncd start`.
- Reload after UCI edits: `/etc/init.d/nrsyncd reload`.
- mDNS TXT check (local advertisement): `ubus call umdns announcements | jsonfilter -e '@["_nrsyncd_v1._udp.local"][*].txt[*]'`.
- mDNS TXT check (network discovery): `ubus call umdns browse | jsonfilter -e '@["_nrsyncd_v1._udp"][*].txt[*]'` (fallback `_rrm_nr._udp` if still present).
- Tests: `tests/run-tests.sh`; installer: `tests/scripts/test_install.sh`.
- Shell lint: `scripts/shellcheck.sh`.

## Chat Outputs

The chat renderer “helpfully” auto-links parts of commands, and it can mangle things that look like Markdown (especially [...]() patterns) when formatting messages.

How to avoid it going forward:

- Put commands on their own line, in a plain code block, with no surrounding punctuation.
- Avoid any [...]() text right next to commands (and avoid smart quoting).
- Keep the command block only the command(s), nothing else.

## Observability & Admin Subcommands

- Logging tag: `nrsyncd` (info/error, debug when enabled).
- Runtime: `/tmp/nrsyncd_runtime` (effective params, skip list, counts, initial_positional_ssids).
- Metrics: `/tmp/nrsyncd_metrics` (cycles, pushes, suppressed, cache hits/misses, uniqueness, failures, per‑iface neighbor*count*\*). Derived ratios via `summary`.
- Subcommands: `mapping`, `mapping_json` (alias `mapping-json`), `neighbors`, `cache`, `refresh`, `diag`, `metrics`, `summary`, `timing_check|timing-check`, `status`, `skiplist`, `reset_metrics`, `version`, `metadata`.
- Readiness probes: `diag` / `timing_check` use `rrm_nr_probe_iface` helper (name reflects upstream ubus method family).
- Metadata inspection: `/etc/init.d/nrsyncd metadata` prints parsed live advertisement.

## Edge Cases / Pitfalls

- Hostapd object mismatch: wait loop until count matches or timeout (exit with error).
- Fractional sleep unavailable: fallback to whole‑second; adaptive retry still capped.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fail-Safe/nrsyncd](https://github.com/Fail-Safe/nrsyncd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
