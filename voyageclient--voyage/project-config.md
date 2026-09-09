---
trigger: always_on
description: This is a fork that targets Ice Cream Sandwich (API 14). Any new feature or code you add MUST run on API 14 — either natively or with an explicit fallback. Never introduce an unconditional dependency on an API that didn't exist at 14 without guarding it.
---

# Platform support (Ice Cream Sandwich / API 14)

This is a fork that targets Ice Cream Sandwich (API 14). Any new feature or code you add MUST run on API 14 — either natively or with an explicit fallback. Never introduce an unconditional dependency on an API that didn't exist at 14 without guarding it.

- Before using a platform API, check its `@RequiresApi` / added-in level. If it's above 14, gate it with `Build.VERSION.SDK_INT` and provide a working path for API 14. A feature that silently no-ops on 14 is not acceptable unless that degradation is deliberate and documented in the code comment.
- Prefer AndroidX/compat wrappers (e.g. `ContextCompat`, `ViewCompat`, `HtmlCompat`) and desugared `java.time`/NIO over raw framework calls, since those already backport behavior to 14.
- Where it costs little, write code so it also works below 14 (down to the lowest the dependency allows) — choose the broadest-compatible API rather than the newest convenient one.
- Don't bump `minSdk`, and don't pull in a library whose own `minSdk` exceeds 14. AndroidX raised its floor 14→19 in Oct 2023 (and 19→21 in 2024), so any new AndroidX artifact must stay on its last minSdk-14 release; the `resolutionStrategy.force` block in the root `build.gradle` pins the stack accordingly.
- When a feature genuinely can't work on 14, the higher-API branch must be isolated behind a version check and the 14 branch must still leave the app usable.

# Vector drawables

`<vector>` is API 21+. Below that only AppCompat can inflate one, and it never gets the chance if the framework resolves the resource first — the platform throws `XmlPullParserException: invalid drawable tag vector`, which surfaces as `InflateException: Error inflating class ImageView` and kills the screen. `vectorDrawables.useSupportLibrary = true` and `setCompatVectorFromResourcesEnabled(true)` do NOT cover these cases.

So, whenever the drawable you are referencing is a `<vector>` (check the file — most `ic_*` in `res/drawable/` are):

- In layouts, never `android:src`, `android:background`, `android:foreground` or `android:drawableStart`/`End`/`Left`/`Right`/`Top`/`Bottom`. Use `app:srcCompat` (on `ImageView`/`ImageButton`) and `app:drawableStartCompat` and friends (on `TextView` and subclasses), and add `xmlns:app="http://schemas.android.com/apk/res-auto"` if the file lacks it. There is no compat attribute for a background — use a PNG or set it from code with `AppCompatResources.getDrawable()`.
- The same ban applies in styles and themes: a `<style>` that sets `android:background` to a vector crashes identically.
- Never nest a vector inside a framework-inflated container drawable — a `<selector>`, `<layer-list>`, `<inset>` or `<ripple>` whose `android:drawable` item points at one. AppCompat's delegate only handles a `<vector>` at the XML root.
- In code, `setImageResource()` / `setBackgroundResource()` / `setCompoundDrawablesWithIntrinsicBounds(int, …)` all go straight to the framework. Use `AppCompatResources.getDrawable(context, R.drawable.x)` and pass the `Drawable`, or `ImageViewCompat`.
- Menu XML `android:icon` is fine — AppCompat's menu inflater already resolves icons through `AppCompatResources`.

The same trap exists in code for any framework method added after API 14 — `View.setBackground` (16), `ImageView.getAdjustViewBounds` (16), `AbsSeekBar.getThumb` (16), the `AssetFileDescriptor`/`Cursor` `Closeable` implementations (19/16). R8 outlines these into `$$ExternalSyntheticApiModelOutline` calls that throw `NoSuchMethodError` on ICS. `im.vector.app.core.extensions.ApiCompatExtensions` holds the shims (`backgroundCompat`, `adjustViewBoundsCompat`, `thumbCompat`, `useCompat`, …) — add to it rather than writing a one-off guard, and note that `background = …` inside an `apply { }` block is the same call with the receiver hidden.

`./gradlew :vector-app:lintRelease` finds all of these; read the `NewApi` entries for `.kt`/`.java` files in `vector-app/build/reports/lint-results-release.xml`. Resource `NewApi` hits are mostly noise (unknown XML attributes are ignored at runtime), and lint misses nothing that desugaring covers, so triage code hits first.

Audit before committing a layout change:

    grep -rn 'android:\(src\|background\|foreground\|drawable\(Start\|End\|Left\|Right\|Top\|Bottom\)\)="@drawable/' --include=*.xml */src/*/res/layout*/

and check whether each hit's drawable file starts with `<vector`.

# Strings

New strings always go into `library/ui-strings/src/main/res/values/donottranslate.xml` with `translatable="false"`. Do not add them to `strings.xml` — that file is the source for translation pipelines and stale entries cause AAPT warnings ("removing resource X without required default value") across every locale.

# Copyright headers

Every file this fork creates gets exactly this header, verbatim, as the first thing in the file:

```
/*
 * Copyright 2026 Voyage Client
 *
 * SPDX-License-Identifier: AGPL-3.0-only
 * Please see LICENSE files in the repository root for full details.
 */
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VoyageClient/Voyage](https://github.com/VoyageClient/Voyage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
