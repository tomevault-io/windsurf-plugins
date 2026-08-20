---
trigger: always_on
description: Read this before changing anything. It is the short brief: what the app is, how
---

# AGENTS.md — working on FitFace Studio

Read this before changing anything. It is the short brief: what the app is, how
to build it, and which mistakes have already been made and fixed here so you do
not re-make them. The reference material lives in [`docs/`](docs/) — start with
[`docs/README.md`](docs/README.md).

## What this app is

An Android app that downloads a Fit3 (SM-R390) watch-face package, losslessly
edits the OPPO-format container inside it, validates the result, and sends those
exact bytes to a paired watch over the accessory + RFCOMM transport.

It is **not** an APK re-signer, does **not** install anything on the watch as an
app, and does **not** rebuild the downloaded package. Personal, experimental,
never shipping to a store. Non-affiliation and the terms of use are in
[`NOTICE.md`](NOTICE.md); the naming rule that follows from it — brand strings
only as literal technical identifiers, never in UI copy — is in
[`CONTRIBUTING.md`](CONTRIBUTING.md#conventions).

| Property | Value |
| --- | --- |
| Application ID | `dev.fitface.studio` |
| minSdk / target / compile | 28 / 36 / 36 |
| JDK | Android Studio's bundled JBR (Java 17) |

## Build and test

```bash
JBR='/Applications/Android Studio.app/Contents/jbr/Contents/Home'
./gradlew -Dorg.gradle.java.home="$JBR" :app:assembleDebug
```

Newer system JDKs break Robolectric (`Unsupported class file major version`), so
always pass the JBR. The full test command, the current baseline and the corpus
setup are in [`CONTRIBUTING.md`](CONTRIBUTING.md#running-the-tests) — run it before
claiming anything passes, and do not restate the counts here or they will drift.

A stale Gradle daemon left over from an earlier Android Studio version fails every
test task with `Failed to exec spawn helper` before a single test runs. `./gradlew
--stop` fixes it; nothing in this repository is wrong when that happens.

Corpus tests are guarded by `Assume`. Do not "fix" a skip by hard-coding a path.

`CanvasIntegrityTest` is the one to extend when a bug is visual rather than structural.
It replays every structural edit over all 99 corpus faces and asserts the canvas still
agrees with itself — the outline matches the artwork filling it, every widget resolves
to an original of the same identity, nothing still on the canvas stops drawing. That
class of bug produces a valid container the watch would accept, so nothing else catches
it. Two of the bugs listed under "Before you touch the format layer" below were found
by these assertions rather than by a crash: the one where a global index was treated
as an identity across a structural edit, and the one where a Static's `+0x20` raster
pointer was left stale when the image section moved.

`libs/*.jar` is gitignored and the two accessory SDK JARs are **not committed** — do
not add them back. The first build on a clean clone needs network to fetch them. See
[`libs/README.md`](libs/README.md).

No doc or build file may depend on a path outside this repository. `analysis/` is a
local working area and is gitignored — findings that matter get written up in `docs/`.

## Module map

Module boundaries and what each one owns are in
[`docs/architecture.md#modules`](docs/architecture.md#modules). The rule that matters
while editing: **Android APIs stop at `:core:data`, `:core:delivery` and the UI
modules.** Binary parsing, protocol framing, CRC calculation, descriptor generation
and install packet encoding stay pure Kotlin and JVM-tested.

## Before you touch the format layer

Everything proven about the container is in
[`docs/bin-format.md`](docs/bin-format.md), and everything the editor is allowed
to change — with the corpus evidence for each rule — is in
[`docs/editing.md`](docs/editing.md). Read the second one at minimum. The rules it
records are not stylistic; each one is there because breaking it made real faces
unopenable or uneditable.

The four that catch people fastest:

* **The panel is not raster 0.** Use `FaceRecordParser.panelSize` and
  `backgroundImage`, never `scanImages(entry).first()`.
* **`drawLeft`/`drawTop` in `:core:model` are the only correct way to derive a
  widget rectangle.** Never call `displayCoordinate` on a widget directly — Badge
  endpoint ordering is handled once, there.
* **No field holds another widget's global index.** An earlier guard based on that
  guess blocked 68% of removals. It is replaced by
  `StructuralEditor.requireSurvivorsUnchanged`. Do not reinstate it.
* **Compare image pointers as record indices, never as raw offsets.** A widget's
  pointers are byte offsets into the image section, so relocating that section rewrites
  them without changing what the widget refers to. `originalWidgetSources` resolves them
  through `payloadKey` before comparing; comparing raw values made every Static on a
  resized face look like a different widget, and faces carrying a row of identical ones
  (00003 has nine) then resolved to nothing and stopped drawing.
* **A global index is not an identity across a structural edit.** `removeWidget`
  renumbers every record after the one it cuts and `appendWidget` puts it back at the
  end, so after a remove-and-restore face `00022`'s seq-10 hour sprite sits at index 10

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [satvikgosai/fitface-studio](https://github.com/satvikgosai/fitface-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
