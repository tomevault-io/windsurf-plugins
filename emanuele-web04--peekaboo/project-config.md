---
trigger: always_on
description: <!-- BEGIN PEEKABOO LIVE SYNC CONTRACT -->
---

<!-- BEGIN PEEKABOO LIVE SYNC CONTRACT -->
## Live Sync Contract — Non-Negotiable

Peekaboo is a local-first SwiftData app whose Mac and iPhone targets synchronize
through the same private CloudKit database. Live two-way sync is a core product
feature, not an optional integration. Never merge, archive, upload, or install a
sync-related change unless every invariant and release check below remains true.

### Mandatory identifiers and environments

- Both targets MUST use bundle identifier `com.emanueledipietro.Peekaboo`.
- Both targets MUST use CloudKit container
  `iCloud.com.emanueledipietro.Peekaboo`.
- Release/TestFlight builds MUST use CloudKit `Production` and production APNs.
- Debug and Local builds MUST use CloudKit `Development` and development APNs.
- `ICLOUD_CONTAINER_ENVIRONMENT` and `APS_ENVIRONMENT` MUST always describe the
  same environment. Never mix Production CloudKit with development APNs.
- Development and Production MUST NOT share a SQLite store. Production uses the
  default SwiftData store; Development uses `development.store`.
- Never point a Debug/Local build at the Production store, and never install a
  development-signed archive as a substitute for the TestFlight Mac build when
  validating production sync.

### Mandatory macOS entitlements

Every macOS configuration (`Peekaboo.entitlements`,
`PeekabooDebug.entitlements`, and `PeekabooLocal.entitlements`) MUST NOT contain
`com.apple.security.temporary-exception.mach-lookup.global-name`, including the
values `com.apple.cloudd` and `com.apple.duetactivityscheduler`.

- Apple rejected Mac build 15 under guideline 2.4.5(i) and explicitly stated
  that these temporary exceptions are not appropriate and will not be granted.
- Do not reintroduce either temporary exception during sync debugging, cleanup,
  security review, or release prep.
- The Mac app MUST keep network client/server, CloudKit container, iCloud
  service, production/development APNs, and App Sandbox entitlements intact.
- CloudKit sync must work through Apple's standard iCloud capabilities without
  direct temporary access to internal Mach services.
- The Mac target MUST link `CloudKit.framework` explicitly. SwiftData and Core
  Data access CloudKit dynamically, which can leave the framework absent from a
  standalone Release binary and prevent App Sandbox from granting standard
  CloudKit service access.

Incident record: Mac build 8 exposed a sync scheduling failure when
`com.apple.duetactivityscheduler` was absent. Build 15 restored both temporary
exceptions, but App Review refused them. Any recurrence must now be fixed using
supported APIs and verified with a real TestFlight two-way sync; the temporary
exceptions must not be restored.

### Persistence and observation invariants

- Keep `NSApplication.shared.registerForRemoteNotifications()` on macOS.
- Keep `UIApplication.registerForRemoteNotifications()` on iPhone so silent
  CloudKit pushes can wake a suspended app and schedule an import.
- Keep the bounded `ProcessInfo` activity assertion around macOS local
  save-to-export and import-to-refresh windows. Peekaboo is an `LSUIElement`
  app and can otherwise enter App Nap while Core Data is still mirroring. The
  assertion must remain event-driven, allow idle system sleep, and retain its
  timeout; never replace it with permanent activity or polling.
- Keep both `NSPersistentStoreRemoteChange` and
  `NSPersistentCloudKitContainer.eventChangedNotification` observation.
- On a completed CloudKit import, replace the long-lived `ModelContext` with a
  fresh context before fetching. A normal fetch on the cached context can keep
  stale values visible and can write them back over imported changes.
- Keep foreground, wake, day-change, time-zone-change, and panel-reveal refresh
  fallbacks. They may refresh local state; they are not a replacement for a
  functioning CloudKit import/export pipeline.
- Do not add aggressive polling. Sync and UI refresh MUST stay event-driven to
  avoid CPU spikes.
- SwiftData models used by CloudKit MUST remain CloudKit-compatible: properties
  need defaults or optionality, and app UUIDs MUST NOT use a SwiftData unique
  constraint that CloudKit cannot enforce.
- CloudKit can contain multiple physical records with the same app-level UUID.
  Deduplicate only for presentation. Never delete an arbitrary duplicate during
  refresh. Mutations and deletion MUST apply to every physical replica of the
  selected app UUID.
- Done-task cleanup MUST use `completedAt` relative to the start of the current
  local day. `updatedAt` must not keep a task completed on a previous day alive.
  A divergent duplicate must prevent destructive cleanup until replicas agree.
- Never delete, reset, migrate, or replace the user's production store or
  CloudKit container as a debugging shortcut without explicit user approval.

### Known failure signatures

Treat these as real failures until disproved:

- `BGSystemTaskSchedulerErrorDomain Code=3`, `updateTaskRequest failed`, or
  repeated `com.apple.coredata.cloudkit.activity.export...` scheduling errors.
  Verify that the Release executable links `CloudKit.framework`, then verify the
  standard iCloud entitlements, installed TestFlight build, active store, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Emanuele-web04/Peekaboo](https://github.com/Emanuele-web04/Peekaboo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
