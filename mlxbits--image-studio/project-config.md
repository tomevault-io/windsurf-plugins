---
trigger: always_on
description: Orientation for coding agents. This is the canonical file; `CLAUDE.md` is a
---

# Agent notes — MLXBits Image Studio

Orientation for coding agents. This is the canonical file; `CLAUDE.md` is a
symlink to it (see "Using this file with your agent" at the bottom).

SwiftUI macOS app that drives [mflux](https://github.com/filipstrand/mflux)
subprocesses to generate images. Xcode 26+, macOS 26 deployment target.

## Build and verify

`project.yml` is the source of truth. **Never edit `.xcodeproj` directly** — it
is generated and your changes will be overwritten.

```bash
xcodegen generate                       # after any project.yml change
scripts/build-python-runtime.sh          # bundled Python runtime → build/python-runtime/ (cached)
python3 -m unittest discover -s scripts/tests   # runtime build-script tests
```

**Before every commit, run CI's lint gate exactly** — both must pass:

```bash
swiftformat --lint --config .swiftformat .
swiftlint lint --config .swiftlint.yml --baseline .swiftlint-baseline.json --strict
```

A plain `swiftlint lint` is not enough: it reports new violations as warnings
among ~139 baselined ones, so they are easy to miss, and CI fails on them under
`--strict`. `type_contents_order` is the usual catch — a property added below
`init` or methods. Run both before building, too: the pre-build SwiftFormat pass
rewrites files and would hide what `--lint` catches.

If `~/MLXBits.xcworkspace` exists (the multi-Studio workspace: Image Studio,
Video Studio, LTX Dataset Studio), you **must** build through it — never
`-project` against the `.xcodeproj`:

```bash
xcodebuild -workspace "$HOME/MLXBits.xcworkspace" \
           -scheme "MLXBits Image Studio" -configuration Debug build
xcodebuild -workspace "$HOME/MLXBits.xcworkspace" \
           -scheme "MLXBits Image Studio" test
```

On a plain clone without that workspace, substitute
`-project "MLXBits Image Studio.xcodeproj"`.

SwiftFormat and SwiftLint also run as pre-build scripts, so a build rewrites
formatting. Lint before you build to avoid a confusing second diff.

## Repo map

```
App/           Entry point + ContentView root layout
Models/        Job models, model catalog, LoRA entries, prompt history/templates
Runner/        Generic JobRunner<Spec> engine, per-family specs, warm-driver controller
Stores/        @Observable app state (settings, per-family job stores, gallery, timings)
Utilities/     Keychain, metadata sidecars, progress parsing, the Python toolchain, caption/scenario LLMs
Views/         SwiftUI, one subdirectory per surface (see below)
Tests/         Swift Testing unit tests — pure logic only, no UI tests
Resources/     Info.plist, entitlements, and the Python drivers shipped in the bundle
project.yml    XcodeGen manifest — source of truth for targets, sources, Info.plist
```

`Resources/mflux_driver.py` and `Resources/scenario_llm_driver.py` are bundled
resources, not build inputs. Changing the NDJSON protocol means changing both
the Python driver and `Runner/DriverProtocol.swift`.

## Naming conventions

Five model families, defined in `Models/ModelFamily.swift`: `flux`, `ideogram4`,
`krea2`, `zimage`, `seedvr2`. Files follow the family name exactly:

| Concern | Path |
|---|---|
| Job model | `Models/<Family>Job.swift` |
| Runner spec | `Runner/<Family>JobRunner.swift` — `enum <Family>RunnerSpec: JobRunnerSpec` plus `typealias <Family>JobRunner = JobRunner<<Family>RunnerSpec>` |
| Job store | `Stores/<Family>JobStore.swift` |
| Params UI | `Views/<Family>/<Family>ParamsPanelView.swift` + `…ParamsPanelState.swift` |
| Preview UI | `Views/<Family>/<Family>PreviewViews.swift` |
| Settings form | `Views/Settings/ModelDefaultsView+<Family>Form.swift` |

Glob for these rather than grepping file contents.

**Three irregularities to know before you search:**

1. **Flux is the unprefixed default.** Its store is `Stores/JobStore.swift` (not
   `FluxJobStore`), its panel state is `Stores/ParamsPanelState.swift`, and its
   views live in `Views/ParamsPanel/`, not `Views/Flux/`. There is no
   `Views/Flux/` directory.
2. **SeedVR2 has no `Views/SeedVR2/`.** It is an upscale action on an existing
   image, not a pickable family, so its UI lives in
   `Views/PreviewPane/SeedVR2PreviewViews.swift` and `SeedVR2UpscaleSheet.swift`.
   It appears in `ModelFamily` only to get its own `GenerationCoordinator` gate
   identity (the OOM guard that serializes it against generative runs).
   `ModelFamily.generative` is the list that excludes it.
3. **Ideogram 4 is much larger than the others.** `Views/Ideogram4/` carries a
   whole bounding-box editor (`BBoxEditorView` plus `+Canvas`, `+Gestures`,
   `+Subviews` extensions) and a caption editor. Start at `BBoxSemantics.swift`
   and `BBoxGeometry.swift` for the model behind that UI.

## Do not search these

Gitignored but present on disk, and expensive to grep:

- `build/`, `DerivedData/` — build output
- `.jscpd-report/` — duplication-checker HTML
- `*.xcodeproj/` — generated from `project.yml`; read the manifest instead
- `docs/screenshots/` — binary PNGs
- `build/python-runtime/` — the bundled Python runtime (~1.6 GB)

## Architecture in one paragraph

`Runner/JobRunner.swift` is a generic engine: `JobRunner<Spec: JobRunnerSpec>`
owns queueing, progress parsing, cancellation, and output handling, while each

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MLXBits/image-studio](https://github.com/MLXBits/image-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
