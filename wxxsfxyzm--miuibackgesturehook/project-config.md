---
trigger: always_on
description: This repository is an LSPosed module for researching Xiaomi/MIUI back gesture behavior.
---

# AGENTS.md

## Project Context

This repository is an LSPosed module for researching Xiaomi/MIUI back gesture behavior.

Current workspace:

```text
D:/code/miui-back-gesture-hook
```

Previous jadx workspace:

```text
D:/code/jadx
```

## Current Goal

The old MiuiHome-side experiment is abandoned.

The current direction is SystemUI-first: find and undo the Xiaomi path where SystemUI delegates gesture/back progress handling to MiuiHome, then restore the AOSP SystemUI/WM Shell back gesture pipeline.

The same-activity
`TYPE_CALLBACK` gesture baseline is currently usable. Preserve that baseline while researching and restoring the remaining AOSP WM Shell remote-animation behavior.

Primary target process:

```text
com.android.systemui
```

Do not reintroduce MiuiHome `BackAnimationAdapter` injection, MiuiHome hand-written
`SurfaceControl.Transaction` animations, or system_server cleanup experiments unless the user explicitly asks to revisit them.

Current remote-animation goal:

- Restore the whole AOSP WM Shell back animation behavior, not only `TYPE_CROSS_ACTIVITY`.
- Use local AOSP reference source at `D:/code/aosp-windowmanager`.
- Checked-in AOSP reference snippets are consolidated under `refs/aosp_back/`, split into `shell/` and `systemui/`.
- Prefer restoring Shell registry/runner/adapter wiring before writing custom Surface animation code.
- `TYPE_CROSS_ACTIVITY` and `TYPE_CROSS_TASK` can be restored from existing Xiaomi Shell animation objects if they exist.
-

`TYPE_RETURN_TO_HOME` depends on launcher registering a back-to-launcher callback through Shell; do not fabricate it without confirming the MiuiHome/SystemUI registration path.

## Current Findings

The loaded Xiaomi code contains AOSP-style SystemUI and WM Shell back components under jadx-renamed packages:

```text
com.android.systemui.navigationbar.gestural.EdgeBackGestureHandler
com.android.wm.shell.back.BackAnimationController
com.android.wm.shell.back.ShellBackAnimationRegistry
com.android.wm.shell.back.CrossActivityBackAnimation
com.android.wm.shell.back.DefaultCrossActivityBackAnimation
```

In jadx output these WM Shell classes may appear as:

```text
com.android.p190wm.shell.back.*
```

Use runtime class names without the jadx numeric package segment in module code:

```text
com.android.wm.shell.back.*
```

The Xiaomi-specific SystemUI bridge to the launcher is:

```text
com.android.systemui.recents.MiuiOverviewProxy
```

Its binder descriptor is:

```text
com.miui.systemui.shared.recents.IMiuiSystemUiProxy
```

Known transaction:

```text
4 -> onGestureLineProgress(float)
```

Current hook blocks this Xiaomi gesture-line progress callback so gesture progress can remain on the SystemUI/AOSP path.

The current module also prevents MiuiHome from adding/showing its back stub window. This is only to keep gesture ownership in SystemUI; it is not the old MiuiHome predictive-back adapter experiment.

The AOSP Shell path to verify in logs is:

```text
EdgeBackGestureHandler.setBackAnimation(...)
EdgeBackGestureHandler.updateIsEnabled()
BackAnimationController.onGestureStarted(...)
BackAnimationController.onThresholdCrossed()
BackAnimationController.onBackNavigationInfoReceived(...)
BackAnimationController.startSystemAnimation()
BackAnimationController.finishBackNavigation(...)
ShellBackAnimationRegistry.updateSupportedAnimators()
```

Current same-activity predictive back finding:

```text
BackNavigationInfo type=4 -> TYPE_CALLBACK
```

For `TYPE_CALLBACK`, Xiaomi/AOSP `BackAnimationController.dispatchOnBackProgressed(...)` suppresses SystemUI-dispatched progress while
`BackNavigationInfo.isAppProgressGenerationAllowed()` is true. That works only when the app receives the raw touch stream and generates progress itself. Because this module uses a SystemUI-owned native input monitor and pilfers the gesture stream, the app does not reliably receive that stream. The module therefore calls:

```text
BackNavigationInfo.disableAppProgressGenerationAllowed()
```

when `BackAnimationController.onBackNavigationInfoReceived(...)` receives `TYPE_CALLBACK`, forcing SystemUI/WM Shell to dispatch
`onBackProgressed` to the app callback.

This must happen before the original `onBackNavigationInfoReceived(...)` body runs. If it happens after
`chain.proceed()`, Shell can still make its initial
`isAppProgressGenerationAllowed()` decisions on the old value, which breaks stricter clients such as Compose predictive-back handlers.

Progress distance mapping:

- Do not use a short fixed visual distance such as `220dp` for app callback progress.
- Current known-good logic baseline is SystemUI native input monitor ownership, early pointer pilfer at `8dp`, fixed
  `48dp` trigger threshold, native `BackPanelController` event dispatch, and release-time invoke/cancel.
- Only the progress denominator should differ from that baseline: use display width instead of
  `220dp`, while keeping the same cancel/commit state machine.
- Do not reintroduce the later v22/v23 experiments that moved trigger ownership to the native panel or intercepted
  `setTriggerBack(false)`; those caused wrong cancellation behavior.

Remote animation registry finding:

- AOSP

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wxxsfxyzm/MiuiBackGestureHook](https://github.com/wxxsfxyzm/MiuiBackGestureHook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
