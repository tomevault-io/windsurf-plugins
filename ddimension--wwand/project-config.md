---
trigger: always_on
description: wwand is an event-driven **ucode** QMI/MBIM connection manager for OpenWrt,
---

# wwand — project guide for Claude

wwand is an event-driven **ucode** QMI/MBIM connection manager for OpenWrt,
replacing the legacy bash QMI dialer. Repo: github.com/ddimension/wwand.
Everything is English. Commit/push only when asked.

## Layout
This repo is the wwand **source tree** (package root = repo root):
`src-ucode/` (core), `io/` (native C module `io/src/wwand-io.c`:
message-oriented cdc-wdm/tty I/O + rmnet netlink helper;
`io/build-target/wwand_io.so` is the cross-built aarch64 module,
`io/build-host/wwand_io.so` the host build used by the tests), `files/`
(netifd shim, init, hotplug, migrate), `tests/`, `tools/`,
`docs/reference.md` (config + ubus API reference).
- **Package definitions live in ddimension/openwrt-repo** (the feed):
  `wwand/Makefile` there builds, from this repo (git source, pinned via
  PKG_SOURCE_VERSION — after pushing here, pin it on the feed's `main` with
  `scripts/bump-source.sh wwand <tag|commit>`, which also derives the version:
  `X.Y.Z` at tag `vX.Y.Z`, `X.Y.Z_pN` after it; the feed's CLAUDE.md has the
  rules, releases go to its `stable` branch only when asked), a **backend-neutral
  base + per-backend split**: `wwand` (daemon/framework/codec/shared core +
  the native `wwand_io.so`, which since 2026-08 ships inside the base package —
  the separate `ucode-mod-wwand-io` package is gone, `PROVIDES` covers old
  configs), `wwand-qmi`, `wwand-mbim` (DEPENDS wwand-qmi), `wwand-ncm`,
  `wwand-mhi` (PCIe/MHI transport + MHI kmods + the wwan-subsystem hotplug;
  backend-neutral, pair with wwand-qmi/-mbim), `wwand-esim`, plus two optional
  DATAPATH add-ons — `wwand-datapath-rmnet_nss` (vendor `qmi_wwan_q`, USB) and
  `wwand-datapath-rmnet_nss_mhi` (vendor `pcie_mhi`, PCIe/MHI) — which adopt the
  QMAP children those drivers register so Qualcomm NSS offload survives.
  Backends do **not** CONFLICTS the stock handlers — wwand coexists with
  uqmi/umbim/comgt-ncm and manages only `proto wwand`.
- **ucode is shipped precompiled to bytecode by default** (production builds),
  built by the **repo-root `CMakeLists.txt`** alongside `wwand_io.so` — one
  compiler invocation per file over **explicit source lists** (no glob as build
  input; a configure-time check fails on list drift, so ADD NEW `.uc` FILES to
  `WWAND_UCODE_MODULES`/`_PROGRAMS`/`_PLAIN` there). Every intra-tree module
  name is passed as `dynlink=` so files compile independently (no ordering;
  imports resolve at runtime via the search path); `-s` keeps the bytecode
  relocatable; a typo'd import is still a compile error. **Graceful fallback:** a
  configure-time capability probe checks the host ucode can actually emit a
  bytecode module with these flags — an older/absent ucode DOES NOT fail the
  build, it ships the ucode SOURCE instead (functional, no bytecode start-up
  win). That is not hypothetical and not a defect: **openwrt-25.12 ships source,
  snapshot ships bytecode**, because 25.12's ucode (2026.01.16) does not know
  `-cmodule` at all — it ignores the flag, compiles the file as a program, and
  `export` is illegal there. Nor is there a way around it: that interpreter
  cannot even LOAD a bytecode module (it reads the file as source and stops at
  the magic), so producing them elsewhere does not help. It flips by itself once
  the branch carries a newer ucode. Note the coupling is to the ucode VERSION,
  not the target: bytecode is architecture-independent (a module from the
  aarch64 package loads in an x86-64 interpreter). Dev opt-out:
  `CONFIG_WWAND_UCODE_SOURCE` (feed menuconfig) / `-DUCODE_PRECOMPILE=OFF`.
  **Invariants (configure fails otherwise): imports namespaced
  (`wwand.codec.tlv`), NEVER relative; no hyphens in module paths (hence
  `codec/mbim_schema/`).** require()-CommonJS shims (top-level `return`,
  `WWAND_UCODE_PLAIN`) stay source; they may import bytecode modules freely.
- LuCI packages moved to their own repos: ddimension/luci-proto-wwand,
  luci-app-wwand (sources only; package defs + wwand-lpac entirely in the
  feed repo).
- **Config: all in `/etc/config/network`** (WireGuard-style: `wwand_modem` /
  `wwand_sim` (per-ICCID override) / `interface proto wwand + option modem` /
  `wwand_globals`). Proto is **`wwand`**. Good citizen: wwand manages ONLY
  `proto wwand` interfaces and the shim registers ONLY that proto — the `qmi`
  name stays uqmi's, and a bare `proto qmi` interface is never adopted. There is
  no switch for it (the former `option takeover` is gone). No `/etc/config/wwand`
  for new installs; migration to the native model is always user-triggered (LuCI
  modem list / `config.migrate_plan` / `/usr/libexec/wwand/migrate`, or the
  example uci-defaults script in `/usr/share/wwand/examples/`).
  Full model in `docs/reference.md`; how to extend in `docs/extending.md`.
- **Zero-config autosetup** (default ON, opt-out `wwand_globals option
  autosetup '0'`): modem appears on an unconfigured box → daemon hotplug
  creates `wwmodem_auto` + `interface wwan0` (default wan firewall zone),
  then ONE-SHOT ICCID/IMSI→APN fill from `apndb.uc` is COPIED into uci
  (marker `option autosetup 1` cleared; uci writers live in main.uc deps
  `autosetup_create`/`autosetup_fill`). HW-verified on the Cudy LT300.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddimension/wwand](https://github.com/ddimension/wwand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
