---
trigger: always_on
description: This file adds repository-specific routing and constraints to the inherited
---

# AGENTS.md

This file adds repository-specific routing and constraints to the inherited
global agent, product-truth, and orchestration rules. It does not restate those
global contracts.

Detailed HoldType behavior lives in `docs/specs/`, selected through
`docs/specs/index.md`.

## Repository Context

`holdtype-swift` is a native macOS menu bar dictation utility. The app records
microphone input, sends audio to OpenAI transcription, and inserts returned
text into the current active app.

The repository contains macOS and in-progress iOS implementation code, tests,
specs, backlog workflows, and automation runbooks.

## Local Document Routing

Read the smallest project-specific file set required by the current action:

- `docs/specs/index.md` is the specification registry. Use it to select the
  exact active product contracts required by the global product-truth gate.
- `SWIFT.md` governs Swift, SwiftUI, AppKit adapter, Xcode project, and test
  changes. For product work, read it after the provisional product-contract
  basis; for behavior-neutral Swift work, read it before opening or editing
  those implementation paths.
- `docs/specs/brownfield-discovery.md` is the current repository map when
  source ownership is unclear.
- `BACKLOG_DEVELOPMENT.md` applies only to explicit backlog work, scheduled
  backlog automation, backlog scripts/runbooks, or backlog-file maintenance.
- `docs/specs/backlog.md` applies only when grooming or selecting product
  areas.
- `docs/agent-tooling.md` applies when choosing Xcode, simulator, device,
  runtime-QA, MCP, or Computer Use tooling.
- `docs/openwhispr_swiftui_codex_tz.md` is fallback source evidence only when
  an initial MVP behavior is not settled by a current contract.
- `docs/openwhispr-reference-retirement.md` records the boundary for the
  removed OpenWhispr snapshot.

Before any iOS Simulator, iPhone Mirroring, or signed physical-device runtime
QA, read and follow `iOS Simulator, Mirroring, And Physical Device QA` in
`docs/agent-tooling.md`.

## Mandatory UI Skill Gate

This gate applies to every interface task: UI design, layout, visual polish,
interaction changes, UI bug investigation, accessibility work, and runtime
visual QA. It applies before inspecting UI implementation code or proposing a
visual solution.

- For macOS UI work, read and use `build-macos-apps:swiftui-patterns`.
- For iOS UI work, read and use `build-ios-apps:swiftui-ui-patterns`. When
  running or debugging the iOS interface, also use
  `build-ios-apps:ios-debugger-agent`; use the other Build iOS Apps skills when
  their specialized surface applies.
- State in the first UI progress update which skill applies and why. Follow the
  selected skill's interaction, state-ownership, component, and verification
  guidance throughout the task.

### Mandatory Computer Use For UI QA

For every macOS or iOS task that changes a visible interface or interaction,
agents must use the [@Computer](plugin://computer-use@openai-bundled) plugin
for the runtime QA pass whenever it is available in the session. The agent
must inspect the actual app, perform the changed action by clicking or keyboard
interaction, and inspect the resulting screen. Do not replace this with
AppleScript, `osascript`, JXA, CGEvent synthesis, or an unattended
screenshot-only check.

An alternative verification path is allowed only when
[@Computer](plugin://computer-use@openai-bundled) is unavailable or cannot
perform the specific interaction after a bounded attempt. Record the concrete
limitation and use the narrowest non-AppleScript fallback.

## Local Apple UI Exceptions

### Narrow Fixes Popup Exception

The user approved one narrow exception on 2026-08-05 for the macOS Fixes
palette and its unavailable-feedback dialog: their presentation shell may use
an AppKit `NSPanel` only to preserve non-activating interaction with a captured
external text target and global click-outside dismissal. Every visible popup
view, its controls, layout, state, and feedback content must remain SwiftUI.
This exception does not permit AppKit in Manage Fixes, editor content, or any
other visible product surface.

### Existing Native Dialog Maintenance

Working AppKit alerts, sheets, and confirmation dialogs, including Quit, may
receive targeted maintenance fixes that preserve their existing focus,
modality, and termination behavior. This maintenance rule does not authorize a
new AppKit surface or a substantial redesign. New or substantially redesigned
visible interfaces must follow the inherited SwiftUI boundary.

## Direct Chat Work Versus Backlog Work

Ordinary user requests in a live chat are direct tasks. Do not create backlog
tasks or run the selector unless one of these conditions applies:

- the user explicitly asks to use, create, select, decompose, groom, archive,
  or execute backlog tasks;
- a scheduled automation or installed runbook identifies the run as backlog
  work;
- the request maintains backlog files, scripts, or runbooks;
- the user and agent explicitly agree to make a long effort restartable
  through committed backlog tasks.

If a direct task needs later follow-up, report it in chat. Create durable
backlog entries only with user approval or when the active automation requires
them.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [holdtype/holdtype-swift](https://github.com/holdtype/holdtype-swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
