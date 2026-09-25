---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

The Gradle workspace is `ARE/`, not the repo root. All Gradle commands must be run from `ARE/`.

- `ARE/are` — the publishable library module (`com.chinalwb.are`, Java only, AGP 8.5.2, minSdk 21 / compileSdk 34, published as `com.github.chinalwb:are`).
- `ARE/app` — the demo app (`com.chinalwb.are.demo`, Java + a little Kotlin, viewBinding enabled). Launcher activity is `IndexActivity`.
- `docs/sdk-production-roadmap.md` — the current plan for hardening the library into a real SDK; check it before large refactors so changes align with the intended direction.
- `AGENTS.md` — repository conventions (commit style, PR expectations); its guidance applies here too.

Requires JDK 17 (AGP 8.5.2). `ARE/local.properties` holds `sdk.dir` and is machine-specific.

## Commands

```bash
cd ARE
./gradlew assemble                 # build both modules
./gradlew :app:assembleDebug       # demo APK
./gradlew :are:build               # compile library + unit tests + package
./gradlew :are:testDebugUnitTest   # JVM unit tests
./gradlew :are:connectedAndroidTest# instrumentation tests (needs device/emulator)
./gradlew lint                     # lint (are module has abortOnError false)

# single test class / method
./gradlew :are:testDebugUnitTest --tests "com.chinalwb.are.UtilPathResolutionTest"
./gradlew :are:testDebugUnitTest --tests "com.chinalwb.are.UtilPathResolutionTest.buildExternalStorageDocumentPath_returnsMatchingVolumePath"
```

JVM unit tests cover logic that can be expressed without framework types (the `Util` paragraph and path helpers). Everything span- or view-shaped needs instrumentation: `SpannableStringBuilder`, `Layout` and `Canvas` all return stubs under the mockable android.jar. Instrumentation tests build editors on `TestHostActivity` (declared in `src/androidTest/AndroidManifest.xml`) — `AREditText` resolves its padding via `Util.getPixelByDp`, which walks the context chain up to an `Activity`, so an application context is not enough.

## Architecture

Everything is built on Android `Spanned`/`Editable` and `android.text.style` spans — there is no WebView and no custom text layout.

### The core loop

`AREditText extends AppCompatEditText` holds a list of `IARE_Style`. Its `TextWatcher.afterTextChanged` calls `style.applyStyle(editable, start, end)` on every registered style for the changed range; `onSelectionChanged` fans out to `IARE_ToolItem.onSelectionChanged` so toolbar buttons update their checked state. So: **a style is only active if it was registered on the AREditText**, via either `setToolbar(IARE_Toolbar)` (pulls `getStyle()` off each tool item) or `setFixedToolbar(ARE_Toolbar)` (pulls `getStylesList()`).

Block-level styles (list, quote, alignment, indent) all resolve their range through `Util.getCurrentCursorLine` / `getThisLineStart` / `getThisLineEnd`. Those are **paragraph** boundaries computed from the text, not `Layout` lines — a `Layout` line is a *visual* line, so reading boundaries off it styled only the fragment of a wrapped paragraph that fit on screen, and returned stale offsets right after an edit (the layout has not been rebuilt yet). The underlying `Util.getParagraph{Index,Start,End,Count}(CharSequence, int)` helpers are pure and unit-tested; `getThisLineEnd` includes the trailing `\n`, matching `Layout#getLineEnd`.

Ordered-list numbers are display-only state, recomputed for the whole document by `ARE_ListNumbering.renumber(Editable)` rather than patched span by span (`Editable#getSpans` returns spans in insertion order, never document order). `reNumberBehindListItemSpans` is kept as a thin delegating wrapper. Numbering restarts at 1 whenever a non-ordered paragraph — plain text or a bullet — interrupts the run.

List items carry a leading `Constants.ZERO_WIDTH_SPACE_STR` marker so an empty item still has a line to draw on; anything that stops a line being a list item has to delete that marker too (`Util.removeZeroWidthMarker`), or it accumulates invisibly and lands in the exported HTML.

`ARE_ABS_Style<E>` is the generic base for character styles. It reflects the span class `E` out of its generic superclass and implements the full checked/unchecked × insert/delete/select matrix of span splitting and merging. New character styles should extend it and implement `newSpan()` rather than hand-rolling span math.

### Two parallel style hierarchies — know which one you are editing

This is the single biggest trap in the codebase. Each feature exists twice:

| | Fixed toolbar | Composable toolbar |
|---|---|---|
| Toolbar | `styles/toolbar/ARE_Toolbar` (LinearLayout, inflates `are_toolbar.xml`, hardcodes every style) | `styles/toolbar/ARE_ToolbarDefault` (HorizontalScrollView, implements `IARE_Toolbar`) |
| Style impl | `styles/ARE_Bold`, `ARE_Italic`, … | `styles/toolitems/styles/ARE_Style_Bold`, `ARE_Style_Italic`, … |
| Entry point | `AREditor` (RelativeLayout wrapping `AREditText` + `ARE_Toolbar`) | `AREditText` + `IARE_Toolbar` assembled by the host |
| Extra layer | — | `styles/toolitems/ARE_ToolItem_*` (view + style pairing, extends `ARE_ToolItem_Abstract`) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chinalwb/Android-Rich-text-Editor](https://github.com/chinalwb/Android-Rich-text-Editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
