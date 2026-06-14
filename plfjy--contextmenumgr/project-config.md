---
trigger: always_on
description: This repository is **Context Menu Manager Plus / ContextMenuMgr**.
---

# AGENTS.md

This repository is **Context Menu Manager Plus / ContextMenuMgr**.

It is a Windows system utility involving a WPF frontend, Windows Service backend, TrayHost, ProbeHost, Windows Registry operations, user SIDs, session IDs, Named Pipe IPC, multi-architecture build artifacts, and multiple privilege/user-context flows.

This file is the main entry point for AI coding agents and human maintainers.

Before analyzing bugs, adding features, refactoring, or changing behavior, read this file first, then read the relevant documents under `docs/`.

Most detailed documents under `docs/` are written in Simplified Chinese. Do not skip them because of language. Read and translate them internally if needed.

---

## 1. Required Reading

Before starting any task, read at least:

1. `docs/ai-maintainer-playbook.md`
2. `docs/process-and-privilege-flows.md`
3. `docs/developer-guide.md`

For module-specific work, also read the matching topic document:

| Task type | Required document |
| --- | --- |
| Classic context menus / registry menu model | `docs/registry-model.md` |
| ShellNew / SendTo / WinX / SpecialMenu | `docs/special-menus.md` |
| Windows 11 modern context menu | `docs/windows11-context-menu.md` |
| Deep Analysis / ProbeHost / Shell Extension probing | `docs/deep-analysis-probehost.md` |
| Frontend UI / WPF-UI / theme / NavigationView / AutoSuggestBox | `docs/frontend-wpf-ui.md` |
| Build, release, installer, multi-architecture ProbeHost | `docs/build-and-release.md` |
| Unclear bugs, user reports, runtime failures | `docs/troubleshooting.md` |
| New agent handoff, fault attribution, pre-change checklist | `docs/ai-maintainer-playbook.md` |

Do not rely only on `README.md`. The README is user-facing. Development decisions must be based on `docs/` and the current code.

---

## 2. Identify the Correct Flow Before Editing

This project is not a single “run as administrator” model. Every bug or feature must first be mapped to the correct process, privilege, and user-context flow.

| Flow | Shape | Main purpose |
| --- | --- | --- |
| Flow A | Frontend -> Backend Pipe -> Backend Service | Normal runtime menu operations, scans, approvals, Win11 blocking, SpecialMenu operations, runtime AutoStart read/write |
| Flow B | Frontend -> UAC runas -> BackendServiceBootstrapper | Install/repair/uninstall/stop service, set service startup mode |
| Flow C | Backend Service -> WTS User Token -> CreateProcessAsUser | Launch TrayHost or Frontend inside the interactive user session |
| Flow D | Frontend -> ProbeHost -> Shell Extension COM | Deep Analysis only; isolated third-party Shell Extension probing |

Before modifying code, answer:

- Which process is involved?
- Which flow should handle this task?
- Does it need the frontend user SID?
- Does it need a session ID?
- Does it touch `HKCU` or `HKEY_USERS\<SID>`?
- Does it require elevated registry writes?
- Does it load or probe a third-party Shell Extension?
- Is this only a frontend UI state problem?

If you cannot answer these questions, stop and read the relevant docs or inspect the code first.

---

## 3. Hard Rules

### User context

- Do not use the service process `HKCU` as the frontend user’s `HKCU`.
- User-scoped registry reads/writes must check whether they need `HKEY_USERS\<SID>`.
- Privilege and user context are different problems. LocalSystem is highly privileged, but it is not the interactive frontend user.
- Win11 user blocked lists, SpecialMenu, AutoStart policy, Restart Explorer, and user-profile paths may require frontend user context.

### Privilege flows

- Normal runtime operations should go through the Backend Pipe. Do not route them through the UAC bootstrapper.
- The UAC bootstrapper is only for service lifecycle and startup-mode operations.
- A Windows Service must not directly show UI. Launch TrayHost or Frontend through the interactive user session.
- Restart Explorer must target the frontend user session. Do not blindly kill all `explorer.exe` processes.

### Frontend UI and WPF-UI

- Frontend UI bugs are not automatically backend, registry, or service bugs.
- For WPF-UI, theme, NavigationView, AutoSuggestBox, Popup, XAML style, or Page/UserControl issues, read `docs/frontend-wpf-ui.md` before changing code.
- Do not fix UI state, template, or binding bugs by changing Backend Service, registry scanning, or privilege flows unless there is evidence the backend is involved.

### SpecialMenu

- ShellNew, SendTo, and WinX are not ordinary `shell` / `shellex` entries.
- For SpecialMenu bugs, inspect `SpecialMenuService` and the related ViewModel first. Do not start by changing `ContextMenuRegistryCatalog`.
- Registry Write Protection and ShellNew ACL Lock are different features and must not be mixed.

### Windows 11 modern context menu

- Windows 11 modern context menu entries are not classic context menu entries.
- Win11 snapshot and block/unblock operations must preserve the correct user context.
- If a Win11 item appears enabled again after refresh, first verify whether the snapshot used the correct `HKEY_USERS\<SID>` blocked list.

### ProbeHost / Deep Analysis

- ProbeHost is not an elevation path. It is an isolation path.
- Do not directly load third-party Shell Extension DLLs inside the Frontend or Backend Service.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PLFJY/ContextMenuMgr](https://github.com/PLFJY/ContextMenuMgr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
