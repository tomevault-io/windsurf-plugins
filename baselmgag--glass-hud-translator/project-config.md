---
trigger: always_on
description: Arabic translation overlay for games that ship without Arabic support. Reads a rectangle of the
---

# Orientation for anyone changing this code

Arabic translation overlay for games that ship without Arabic support. Reads a rectangle of the
screen, OCRs it, translates it, draws the result back over the game in a separate window.

Start here, then read [`docs/BRIEF.md`](docs/BRIEF.md) for why the project is shaped this way and
[`PROJECT_PLAN.md`](PROJECT_PLAN.md) for the type contracts and schemas.

The later multilingual product discussion is recorded in
[`docs/MULTILINGUAL_VISION_REPORT.md`](docs/MULTILINGUAL_VISION_REPORT.md). It explains the proposed
direction and the research behind it; it is context for discussion, not the current implementation
contract or an instruction to replace the design documents above.

## The constraint behind most of the design

I develop on macOS. Live testing happens on a borrowed Windows laptop that is rarely available. So
the whole codebase is arranged to run as much as possible without Windows, and to keep the part
that genuinely needs it as small as I could make it.

```
runs on macOS/Linux   all logic, prompts, glossary, cache, OCR, the entire UI,
                      full pipeline replay against recorded screenshots
needs Windows         live screen capture, global hotkeys, click-through,
                      DPI handling, a real game
```

If you contribute, you can do almost everything without a Windows machine.

## Commands

```bash
dotnet build
```

Note it's plain `dotnet build`, not `dotnet build -f net10.0`. That flag is per-project, and at
solution level it tries to force `net10.0` onto the Windows-only projects and fails.

```bash
dotnet test
```

611 tests, all runnable on macOS and Linux.

```bash
dotnet run --project tools/Replay -- --no-cache
```

Replay is the main development loop and the fastest way to understand the system. It pushes
recorded PNGs through the exact `TranslationPipeline` the overlay uses and prints every stage.
Flags: `--provider stub|gemini|groq|ollama|all`, `--profile <id>`, `--frames <dir>`, `--no-skip`
to bypass change detection, `--generate-frames` to rewrite the sample corpus.

```bash
dotnet run --project src/GlassHudTranslator.App -f net10.0 -- --stub
dotnet run --project src/GlassHudTranslator.App -f net10.0 -- --render-test --render-test-out /tmp/render.png --exit-after-render
dotnet run --project src/GlassHudTranslator.App -f net10.0 -- --toolbar-test --toolbar-test-out /tmp
```

Run `--render-test` after touching anything to do with fonts, text layout or the Avalonia version.
It renders the cases where Arabic layout usually breaks and tells you whether the bundled font
actually loaded rather than the OS quietly substituting one.

Run `--toolbar-test` after touching `Icons`. The icons are SVG path strings parsed at runtime, so a
typo is an exception thrown while the window is being built — on a user's machine, at startup, with
no compiler and no unit test able to see it. This draws the real strip, simple and expanded, to two
PNGs. If they render here they parse everywhere.

Run `--failure-test --failure-test-out <dir>` after touching `StartupFailureWindow` or anything on
the startup path — it renders the startup-failure window with a staged exception. Run
`--wizard-test --wizard-test-out <dir>` (optionally `--wizard-test-lang ar`) after touching
`FirstRunWizard` — it walks all four steps for the camera. Both flags exist for the same reason:
these are the windows that only ever appear on a stranger's machine at the worst possible moment,
and whose own failure to build is the most expensive crash the app can have.

## Layout

```
src/GlassHudTranslator.Core/     net10.0                  all logic, all tests
src/GlassHudTranslator.Interop/  net10.0-windows          P/Invoke declarations, no logic
src/GlassHudTranslator.Windows/  net10.0-windows          Win32 impls (untested on hardware)
src/GlassHudTranslator.App/      net10.0;net10.0-windows  Avalonia UI
tools/Replay/                          net10.0                  headless harness
profiles/<game>/                       per-game data, no code
data/models.json                       provider and model config
```

CI is three workflows: `build.yml` (tests on ubuntu at 1x billing), `release.yml` (tag `v*` →
public zip), and `publish-windows.yml`, which both of the others call so that the artifact a
release ships comes from exactly the steps that have been running green on every commit.

A useful thing to know: `net10.0-windows` **compiles on macOS**. The TFM only applies
`[SupportedOSPlatform("windows")]`, which is an analyzer contract, not a build requirement. It's
`UseWPF`/`UseWindowsForms` that would make a Windows host mandatory, and this project uses neither.
That's why a typo in the Win32 layer gets caught locally instead of on the borrowed laptop.

## Rules that look like style but are correctness

**Never set an explicit `LineHeight` on Arabic text.** Arabic hangs marks below the baseline —
kasra, the dot under `ج`, the two dots under a final `ي`. A tight line box clips them silently, and
clipping those two dots turns `ي` into `ى`, which is a different letter and often a different word.
Measured at 26px with the bundled font: `LineHeight` 40 and 44 clip, 48 and above are fine, natural

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BaselMGAG/glass_hud_translator](https://github.com/BaselMGAG/glass_hud_translator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
