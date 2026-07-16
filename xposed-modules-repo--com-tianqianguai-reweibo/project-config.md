---
trigger: always_on
description: LSPosed/Xposed module (legacy `de.robv.android.xposed.*` API, **not** libxposed v2). Hooks into three target apps to modify feed layout and remove ads.
---

# AGENTS.md — ReWeibo

## What this is

LSPosed/Xposed module (legacy `de.robv.android.xposed.*` API, **not** libxposed v2). Hooks into three target apps to modify feed layout and remove ads.

| Target | Package | Hook class | Function |
|--------|---------|------------|----------|
| 微博 | `com.sina.weibo` | `WeiboFeedHook` | Reverse feed + auto-scroll to top |
| Share | `com.hengye.share` | `ShareFeedHook` | Auto-scroll down to "上次阅读" marker + "显示更多" auto-click |
| 微博轻享版 | `com.weico.international` | `WeiboLiteHook` | Remove splash + timeline ads + reverse feed + auto-scroll |

Entry point: `MainHook` implements `IXposedHookLoadPackage`, dispatches by `lpparam.packageName`.

## Build & deploy

```bash
.\gradlew.bat assembleRelease
```

APK output: `app/build/outputs/apk/release/app-release.apk`

Install + restart target (example for Weibo):
```bash
adb -s 192.168.6.17:5555 install -r app/build/outputs/apk/release/app-release.apk
adb -s 192.168.6.17:5555 shell am force-stop com.sina.weibo
adb -s 192.168.6.17:5555 logcat -c
adb -s 192.168.6.17:5555 shell monkey -p com.sina.weibo -c android.intent.category.LAUNCHER 1
```

Device: `192.168.6.17:5555` (Xiaomi, Android 16). Reconnect with `adb connect 192.168.6.17:5555` if dropped.

**When testing, simulate interactions yourself via adb.** Don't ask the user to tap buttons. Use `adb shell input tap x y` for taps, `adb shell input swipe` for scrolls. First confirm the target app is in the foreground with `adb shell dumpsys activity activities | grep mFocusedApp`.

## Debugging

- **Use persistent file logging, NOT logcat buffer.** `XposedBridge.log()` goes to logcat which can overflow or get cleared. For durable logs, write to target app's internal storage via `context.getDir()` or `context.getFilesDir()`. Example: `new File(context.getFilesDir(), "reweibo.log")`.
- ShareFeedHook: `/data/data/com.hengye.share/files/reweibo_share.log`
- WeiboLiteHook: `/data/data/com.weico.international/files/reweibo_weico.log`
- To read persistent logs: `adb shell cat /data/data/<package>/files/<logfile>`

## Xposed config

- `assets/xposed_init` → `com.tianqianguai.reweibo.MainHook`
- `AndroidManifest.xml` meta-data: `xposedmodule=true`, `xposedminversion=82`
- Scope: `res/values/arrays.xml` `xposed_scope` array
- Dependency: `compileOnly(files("libs/xposed-bridge-api.jar"))` — **compileOnly**, not implementation

After changing scope or xposed_init, user must re-enable module in LSPosed manager and restart target app.

## Architecture

**Single module, 7 Java files, no frameworks.** All hooks use `XposedHelpers.findAndHookMethod` / `XposedHelpers.findAndHookConstructor`. No Kotlin, no AndroidX Room/DataBinding in the module itself.

### FloatingButton

- Primary: `WindowManager` + `TYPE_APPLICATION_PANEL` + `activity.getWindow().getDecorView().getWindowToken()` — no `SYSTEM_ALERT_WINDOW` needed
- Fallback: `activity.addContentView()` — works for Weibo/Share, but Weico's view hierarchy sometimes intercepts touches
- Hook `Activity.onResume` to attach, `sLastActivity` dedup prevents re-adding
- Double-tap detection: 500ms window, `SystemClock.elapsedRealtime()`, `onTouchListener(return true)`
- Single-tap stops scrolling: 500ms delayed callback, cancelled if double-tap detected. `SingleTapAction` interface

### WeiboFeedHook

- `onLayout` hook on `RecyclerView` fires intermittently for main process — filter by adapter class name containing `RecyclerViewAdapter`
- `setLayoutManager` hook fires on background thread — `postDelayed` callbacks may find adapter already consumed
- Auto-scroll: `smoothScrollToPosition` triggers `onScrolled` → load-more. `scrollToPosition` does NOT.
- Idle-count polling: 1.5s interval, 30 checks for first-time, 10 for subsequent opens
- SharedPreferences flag via `Application.onCreate` context

### ShareFeedHook

- Share's `ShareRecyclerView` extends `OOo0oO` (obfuscated), NOT standard `RecyclerView`
- `RecyclerView.onLayout` hook fires for the inner RV (standard RecyclerView)
- `setLayoutManager` hook on `RecyclerView` fires when `CustomLinearLayoutManager` is assigned
- `O000000o(true)` on `CustomLinearLayoutManager` = `setReverseLayout(true)`
- "显示更多" button: `FrameLayout` (not `TextView`), hook `View.setOnClickListener` to capture listener
- Listener gets recycled by RecyclerView — must re-capture on each bind, call `onClick(null)` async via Handler
- "上次阅读到这里，点击刷新" marker: search visible TextViews during scroll for "上次"/"阅读"/"刷新"/"已读". Stop scrolling when found. If not found, scroll to bottom.
- Stable check: 10 rounds if count>500, 100 rounds if count>100, then scroll to top

### WeiboLiteHook

- Ad removal: hook `LogoActivity.doWhatNext` → "main", `RxApiKt` lambdas + `isWeiboUVEAd` + `Setting.load*`
- Lambda names vary by version (6.1.7/6.2.6/6.3.8) — default uses 6.2.6 names
- Feed reversal: same as WeiboFeedHook pattern (setReverseLayout + onLayout + auto-scroll)
- RV captured via `setLayoutManager` hook, filter by adapter class name containing `TimelineAdapter` (NOT `MainProfileAdapter` which is drawer items)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xposed-Modules-Repo/com.tianqianguai.reweibo](https://github.com/Xposed-Modules-Repo/com.tianqianguai.reweibo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
