---
trigger: always_on
description: handles; the debug signing materials are the public OpenHarmony samples,
---

# Shadowsocks for HarmonyOS NEXT — Agent Guide

## Project overview

A native Shadowsocks client for **HarmonyOS NEXT (5.x)**, which has no Android
runtime. It is a subproject of the shadowsocks-android repository (this
directory is `harmony/` in that repo) and shares the same Rust core
(`shadowsocks-rust`) with the Android app through a C ABI + NAPI bridge.

- **App layer**: ArkTS/ArkUI, Stage model, API 12+ (`compatibleSdkVersion
  5.0.0(12)`), bundle `com.github.shadowsocks.harmony`, version 5.3.5,
  GPL-3.0-or-later.
- **Native layer**: the Rust crate `native/sslocal-ffi` path-depends on
  `../../../core/src/main/rust/shadowsocks-rust/crates/shadowsocks-service`
  (i.e. `<repo-root>/core/...` — the checkout one level above this directory
  must contain the `core/` module for any Rust build to work).
- **Bridge**: a C++ NAPI shim (`entry/src/main/cpp/napi_init.cpp`) is compiled
  by CMake into `libsslocal.so`, linked against the Rust staticlib
  `libsslocal_core.a`.

## Repository layout

```
AppScope/app.json5            application-level config (bundle name, version)
entry/                        main (and only) HAP module
  build-profile.json5         stageMode; external native build via CMake, arm64-v8a only
  src/main/
    module.json5              EntryAbility + SsVpnExtensionAbility (type: vpn),
                              permissions: INTERNET, GET_NETWORK_INFO
    ets/entryability/         EntryAbility.ets — UIAbility entry; registers the
                              ability context in AppStorage (see AppContext.ets)
    ets/pages/                Index.ets (profile list + connect + stats bar),
                              ProfileEdit.ets (profile form, method/route/plugin
                              pickers), Subscription.ets (subscription management)
    ets/model/                Profile.ets (SIP002 ss:// parsing, config
                              serialization), ProfileStore.ets (multi-profile
                              list + selection), Subscription.ets (subscription
                              fetch/parse: base64 ss:// lists and SIP008 JSON),
                              Routes.ets (policy-route constants ↔ ACL files),
                              TrafficStats.ets (cross-process stats bridge),
                              JsonStore.ets (JSON-file persistence), AppContext.ets
    ets/vpnability/           SsVpnExtensionAbility.ets — VpnExtensionAbility
                              that drives the native core in tun mode; deploys
                              the route's ACL file and hosts the flow-stat
                              endpoint
    resources/rawfile/acl/    ACL files for policy routes, copied verbatim from
                              shadowsocks-android's core/src/main/assets/acl
    cpp/napi_init.cpp         NAPI shim; exports the libsslocal.so module
    cpp/CMakeLists.txt        builds libsslocal.so, links libsslocal_core.a
    cpp/types/libsslocal/     ArkTS typings (Index.d.ts) for libsslocal.so
  src/test/                   ArkTS unit tests (hypium): ss:// URL parsing
                              (incl. SIP003 plugin query), SOCKS and tun config
                              serialization (incl. ACL injection, plugin fields),
                              subscription body parsing
  src/ohosTest/               on-device tests exercising the NAPI surface
                              (including startTunFd) on emulator/device
native/
  sslocal-ffi/                Rust crate: C ABI over shadowsocks-service
    src/lib.rs                extern "C" API: sslocal_start, sslocal_start_tun_fd,
                              sslocal_stop, sslocal_is_running, sslocal_last_error,
                              sslocal_version; single-instance model; intercepts
                              SIP003 plugin fields before handing the config to
                              shadowsocks-service
    src/plugin.rs             SIP003 plugin dispatch + loopback forwarder
    src/obfs.rs               simple-obfs http/tls client wrappers, vendored
                              from meow-rs (GPL-3.0)
    src/v2ray_plugin.rs       v2ray-plugin ws(+tls) client, adapted from
                              meow-rs on top of the meow-transport crate
    tests/e2e.rs              in-process server + sslocal via the C ABI + SOCKS5
                              round-trip through the encrypted tunnel
    tests/e2e_plugin.rs       same, with the connection obfuscated through the
                              in-process simple-obfs plugin (fake obfs server
                              shim in front of the in-process ssserver)
    examples/net_helper.rs    helper binary used by the tun e2e
  build-ohos.sh               cross-compile the Rust core for
                              aarch64-unknown-linux-ohos; installs the staticlib
                              into entry/libs/arm64-v8a/
  sign-hap-debug.sh           ad-hoc debug signing with the OpenHarmony sample
                              signing materials (no Huawei account)
  ohos-cc-wrapper.sh          zig cc shim so `cargo check` works without the
                              OHOS NDK (compile-only; final linking needs the SDK)
  tun-e2e-linux.sh            tun packet-routing e2e (Linux + root)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shadowsocks/shadowsocks-ohos](https://github.com/shadowsocks/shadowsocks-ohos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
