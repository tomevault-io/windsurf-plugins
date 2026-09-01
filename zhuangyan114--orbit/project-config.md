---
trigger: always_on
description: - This is a Windows VS Code extension. The supported target access stacks are J-Link (native C++ helper or in-process `koffi` legacy channel) and CMSIS-DAP/DAPLink (dedicated native C++ helper over WinUSB v2 or HID v1). Do not add OpenOCD, a GDB server, Ozone GUI automation, or `JLink.exe` as a normal debug-control path.
---

# Orbit for VS Code - Agent Guide

## Scope and Working Tree

- This is a Windows VS Code extension. The supported target access stacks are J-Link (native C++ helper or in-process `koffi` legacy channel) and CMSIS-DAP/DAPLink (dedicated native C++ helper over WinUSB v2 or HID v1). Do not add OpenOCD, a GDB server, Ozone GUI automation, or `JLink.exe` as a normal debug-control path.
- The worktree may already contain user changes. Preserve them. Do not revert, overwrite, stage, or otherwise change unrelated files.
- Source is under `src/` and `native/`; do not manually edit generated `dist/` bundles.
- Before changing a debugging bug, load `.agent/skills/ozone-debug-fix/SKILL.md`.
- Before inspecting, changing, diagnosing, or verifying any DAPLink/CMSIS-DAP path, also load `.agent/skills/daplink-debug-fix/SKILL.md`.

## Build and Test

- `npm install` installs the locked dependencies (`package-lock.json` is committed).
- `npm run build` bundles `dist/{extension,debugadapter,webview,timeline,watch}.js`.
- `npm run typecheck` runs `tsc --noEmit`; `npm test` runs Vitest.
- `npm run build:native` builds `out/native/win32-x64/orbit-jlink-helper.exe`.
- `npm run test:cpp-channel:mock` exercises the helper channel against the mock DLL. It is not hardware validation.
- `npm run build:native` also builds `out/native/win32-x64/orbit-cmsis-dap-helper.exe`; `npm run test:cmsis-dap:mock` and `npm run test:cmsis-dap:algorithm` cover the CMSIS-DAP protocol/helper and Flash Algorithm paths. They are not hardware validation.
- Watch/Timeline and stopped-state DAP changes have focused coverage in `src/debug/dap-session-realtime-variables.test.ts`, `src/debug/dap-session-scopes.test.ts`, `src/ozone-backend/native-scheduler.test.ts`, and `src/utils/watch-expression-validation.test.ts`; run the focused tests before the full suite when iterating on those paths.
- `npm run watch`, `npm run dev`, and `npm run mcp` respectively watch bundles, launch Extension Development Host, and start the local MCP client.
- Do not run target-mutating hardware commands without explicit user authorization.

## Process and Native Boundary

- Extension host (`src/extension.ts`) and DAP adapter (`src/debugadapter.ts`) are separate processes with separate backends. The DAP entrypoint communicates over stdio using DAP `Content-Length` frames; it must remain free of `vscode` imports in `src/debug/dap-session.ts`.
- `native/jlink-helper/src/main.cpp` is a Windows child process with the J-Link DLL loaded inside it. `ExperimentalCppJLinkChannel` owns its JSON-lines protocol, lifecycle, and process exit. Keep protocol version/capability changes synchronized with `src/ozone-backend/cpp-jlink-channel.ts`.
- `native/cmsis-dap-helper/src/main.cpp` is the separate Windows CMSIS-DAP child process. `CmsisDapHelperClient` and `CmsisDapTargetChannel` own its JSON-lines protocol, HID/WinUSB lifecycle, SWD/DP/AP access, Cortex-M control, Flash Algorithm execution, memory transport, and process exit. Keep protocol and capability changes synchronized with `src/ozone-backend/cmsis-dap-helper-channel.ts` and `src/ozone-backend/session-target-channel.ts`.
- The legacy channel loads `JLink_x64.dll` through `koffi`. Preserve the existing J-Link safety rules: six slot-indexed hardware breakpoints, no `ExecCommand("SetBP ...")`, no new `readMemoryU32` register/stack/local flows, and existing synchronization after run/step.
- Do not use `JLINK_Close()`/open as an owner switch. In particular, `JLinkDLL.disconnect()` must not close the DLL just to reconnect.

## One Target Owner per Debug Session

- `SessionTargetSelector` is the sole physical target-owner selector. A session has exactly one owner: `jlink-native`, `jlink-legacy`, or `cmsis-dap`.
- `probe: 'cmsis-dap'` creates only the CMSIS-DAP helper owner. It must never construct or fall back to a J-Link native/legacy owner or a second CMSIS-DAP helper. An established CMSIS-DAP owner loss ends the session after disposal; reconnect through a new session.
- When the native path is preferred, create and connect the helper first. Construct the legacy owner only after a failed native owner has fully disposed and its process has exited.
- Legacy fallback is allowed only when native startup fails or the current native owner reports `NativeOwnerLost`. The selector disposes the failed native process, reconnects legacy with the saved config, clears/restores tracked breakpoint slots, then publishes the new owner. Any failed fallback leaves no owner.
- Native source-level step APIs are unavailable on the legacy owner. Do not silently emulate them through a second DLL owner.

## Command, DAP, and Runtime Routing

- `OzoneBackend` dispatches the `OzoneCommand` union. Every `OzoneCommandResult` consumer must test `.ok` before reading `.data` or using success-only fields.
- DAP owns target access while an `orbit` debug session is active; legacy `ozone` is only a compatible session-type alias. `SessionManager` must not connect its extension-host backend in that state.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhuangyan114/Orbit](https://github.com/zhuangyan114/Orbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
