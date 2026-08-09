---
trigger: always_on
description: dasImgui is the daslang binding + boost-v2 wrapper layer for [Dear ImGui](https://github.com/ocornut/imgui), **in-tree at `modules/dasImgui/`** and built by default (root CMake option `DAS_IMGUI_DISABLED`, default `OFF`; needs the in-tree dasGlfw + dasClipboard). It ships the C++ native binding (`bind/`, `src/`; also a `dasModuleImgui.shared_module` for DLL builds), the boost-v2 wrapper layer (`widgets/` — `[widget]` / `[container]` / `with_*` macros, telemetry, default-on lint), the `imguiApp` 
---

# dasImgui module instructions

dasImgui is the daslang binding + boost-v2 wrapper layer for [Dear ImGui](https://github.com/ocornut/imgui), **in-tree at `modules/dasImgui/`** and built by default (root CMake option `DAS_IMGUI_DISABLED`, default `OFF`; needs the in-tree dasGlfw + dasClipboard). It ships the C++ native binding (`bind/`, `src/`; also a `dasModuleImgui.shared_module` for DLL builds), the boost-v2 wrapper layer (`widgets/` — `[widget]` / `[container]` / `with_*` macros, telemetry, default-on lint), the `imguiApp` (windowed GLFW+GL) and `imguiAppHeadless` (no display, real ImGui ctx, CPU font atlas) harness backends, and examples under `examples/`.

The old standalone repo (borisbat/dasImgui) is archived with full history. `daspkg` recognizes `require_package("dasImgui")` as in-tree and reports *part of this daslang tree — nothing to install*; in-repo example `.das_package` manifests do NOT declare it (`daspkg release` — native and wasm — discovers module archives from the compiled require chain via `daslang -exe --list-shared-modules`; the manifest `dependencies()` section drives only the install flow, which has nothing to install for an in-tree module).

## Locations

- Module source: `modules/dasImgui/` (`bind/`, `src/`, `widgets/`, `examples/`, `utils/`)
- Tests: `modules/dasImgui/tests/` — nightly CI lane `.github/workflows/nightly_imgui.yml`; see `modules/dasImgui/tests/README.md`
- Docs: stdlib section `doc/source/stdlib/sec_imgui.rst` (+ generated pages), tutorials `doc/source/reference/tutorials/imgui/*.rst` (see Documentation below)
- Skills: **repo root** `skills/imgui_*.md` (see table below)
- Recordings: intermediates in `doc/source/_static/tutorials/` (gitignored); MP4 deliverables on the rolling `docs-assets` GitHub release (see Recordings below)

## Skill files (REQUIRED)

| Skill file | Read BEFORE... |
|---|---|
| `skills/imgui_ui_debugging.md` | Diagnosing/fixing ANY UI or interaction bug (also mandated by the root CLAUDE.md) |
| `skills/imgui_playwright.md` | Writing/editing any `modules/dasImgui/tests/test_*.das` or `record_*.das` driver — the **async rule** (gate on the effect, not a frame/sleep guess), the `wait_*` family, one-host-per-9090 |
| `skills/imgui_recording.md` | Writing/editing any `record_*.das` driver — pacing constants, workflow, APNG→MP4 conversion |
| `skills/imgui_migration.md` | Migrating v1 daslang+imgui code (`require imgui/imgui_boost`, raw `NewFrame()`/`Begin()`) to v2. Read when you hit IMGUI001 / IMGUI002 |
| `skills/imgui_application.md` | Structuring a long-running dasImgui app (init/update/shutdown lifecycle, heap ownership) |

## Build

Builds with the normal daslang build (`cmake --build build --config Release`); targets `dasModuleImgui`, `imguiApp`, `imguiAppHeadless` exist for DLL/shared-module consumers (artifacts land in `modules/dasImgui/`, not the build dir). Stop any running `daslang-live` / `imguiApp` consumers before rebuilding — on Windows the OS holds locks on loaded DLLs.

## Binding regen (`bind/bind_imgui.das`)

The raw binding under `src/` is GENERATED. After changing the binding surface or bumping the imgui version, regenerate and commit:

```bash
./bin/daslang modules/dasImgui/bind/bind_imgui.das
git diff -- modules/dasImgui/src/     # commit the changes
```

Needs the dasClangBind/libclang stack — read `skills/clang_bind_build.md` first. CI freshness gate: `build.yml`'s mingw nightly worker runs the self-binder and fails on a dirty `modules/dasImgui/src/`.

**imgui version pin:** v1.92.6-docking, fetched via CMake FetchContent at build time. Bumping it means re-running the self-binder and committing the regenerated `src/`.

## Module resolution

- `require imgui/<name>` resolves via the module's `.das_module` `register_native_path` entries; the in-tree project-root scan picks them up. Run from the repo root; recipes pass `-project_root .` where needed (e.g. record drivers).
- `require` only resolves **siblings** of the calling file's directory and the registered native paths. **No `..`/absolute-from-root forms.** Files that need both `imgui/*` and a sibling module must live in the sibling's directory.
- If a sibling name collides with an `imgui::` builtin (`ShowAboutWindow`, `ShowStyleEditor`, …), qualify at the call site: `about::ShowAboutWindow()`.

## Examples layout (`modules/dasImgui/examples/`)

- `features/` — small focused demos, one wrapper per file (~20-80 LOC); drive `[test]` smokes in `modules/dasImgui/tests/test_<name>.das`. `harness_*` lifecycle.
- `imgui_demo/` — daslang port of `imgui_demo.cpp`: per-scene modules consumed by `imgui_demo.das`; `main.das` is the live-reload entry; `harness_<scene>.das` for headless smokes + recordings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaijinEntertainment/daScript](https://github.com/GaijinEntertainment/daScript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
