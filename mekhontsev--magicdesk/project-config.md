---
trigger: always_on
description: Re-read this file after context compaction or session recovery before continuing
---

# Repository Instructions

Re-read this file after context compaction or session recovery before continuing
repository work. Preserve unrelated uncommitted changes; Git history changes
require the user's authorization.

## Device Support Work

The MagicDesk APK targets Android 14 / API 34 and newer. Managed Desktop
requires Android 15 / API 35 and newer. Preserve both paths: newer Desktop
APIs must not become startup requirements for automation or independent tools.
Do not add support below API 34 or change either minimum without an explicit
decision. Build configuration is the installability contract; a successful
build is not proof of compatibility with an untested Android release.

## Runtime Layers

Before changing shared startup or service prerequisites, read the Runtime
Layers section of `docs/architecture.md` and `docs/runtime-api-levels.md`.
MCP, files, content, profiles, shell execution and Termux sessions are shared
services, not Desktop-owned features. Ordinary built-in Activity placement and
virtual-display creation must not acquire HOME, provision Desktop, initialize
its task/session coordinators, or require a WMShell Desktop backend.
Display input is a shared, explicitly acquired service. Opening an application
does not claim input. Desktop acquires it after preparation; manual selection
can control another display without Desktop or its shortcut filter.

Use `ToolApplications` and `ToolLaunchTarget` for built-in placement and
`DisplayOperations` for display resources. A display, its viewer and a Desktop
session have independent lifetimes. Close Desktop does not implicitly remove
its display. Closing or detaching an ordinary terminal window retains its PTY.
An explicitly managed tmux window releases only its client PTY; tmux owns the
server session and programs. MCP is an authorized adapter to these services, not their owner.
Keep profile-scoped application identities and storage boundaries intact.

Embedded X11 is also a shared service: the selected Termux environment supplies
programs, while the fork owns the X server/protocol and rendering. Android hosts
borrow outputs; whole-desktop viewer closure retains the session, whereas an
individual-client host requests that client's closure. Keep clipboard, drag URI
grants and Android placement in the host, not the native renderer. Read
`docs/x11.md` and the fork's `docs/embedding.md` before changing this boundary.

Do not retain obsolete internal APIs, persisted-data formats or MCP protocols
solely for backward compatibility unless explicitly requested. Remove replaced
paths instead of adding migration layers; this does not relax the supported
Android release boundaries.

`RuntimeCapabilities` owns service prerequisites, separately from MCP client
grants. Keep the full MCP tool catalog discoverable; unavailable operations
must fail explicitly without disabling independent services or changing system
state. On API 34, reject Desktop and its self-tests before display preparation
or HOME changes. Prefer ordinary app APIs when sufficient; use the shared
privileged service otherwise. UID 2000 remains the baseline. Shizuku and direct
`su` are startup transports, not alternative feature implementations. Keep
backend selection and the independent force-UID-2000 policy at startup; never
silently elevate or switch identity for an individual operation. Root is not
a product requirement. Read `docs/privilege-modes.md` before changing that boundary.

## Platform Changes

Before changing platform, display, window, input, launcher, or cleanup behavior,
read `CONTRIBUTING.md`, `docs/ai-assisted-device-porting.md`, and the relevant
sections of `docs/architecture.md`, `docs/compatibility.md`, and
`docs/automation.md`.

Establish a current-`main` baseline from a complete compatibility report and an
exact reproduction before editing. Classify the owning boundary first. Keep one
APK and one codebase: prefer shared Android behavior and runtime capabilities;
isolate genuine firmware behavior in a focused `PlatformExtension`, SoC
behavior in `SocDisplayModeBackend`, display lifecycle in the existing four
drivers, and window policy in the existing transition gateway. Do not add model
checks, fixed runtime delays, coordinate-based production actions, root
requirements, or device-specific forks.

Keep Android-release differences in `FrameworkRuntime` and its focused
adapters. `FrameworkWindowingApi` is the only owner of hidden
`WindowContainerTransaction` primitives; `FrameworkWindowingCompat` owns
release-dependent semantics and polyfills; `HiddenTaskApi` owns raw task
members; `FrameworkTaskSnapshotSource` publishes typed Binder snapshots; and
`FrameworkInputWindowObservationSource` owns SurfaceFlinger input-window
commit events, while `FrameworkInputSnapshotSource` owns one-shot
InputDispatcher snapshots. Callers must
not reflect these APIs, pass raw framework member names, or introduce text task
queries themselves. An unavailable framework observation remains unknown; do
not synthesize a value that can be mistaken for an application request. Use
the debug-only
`MAGICDESK_FRAMEWORK_OVERRIDE` profile to exercise older semantics on a newer
device. Framework task observation belongs to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mekhontsev/magicdesk](https://github.com/mekhontsev/magicdesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
