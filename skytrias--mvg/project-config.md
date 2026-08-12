---
trigger: always_on
description: MVG's source is authored for people and coding agents. This guide is the
---

# AGENTS.md

MVG's source is authored for people and coding agents. This guide is the
operator manual: where to find the source, how to make a safe change, and how
to turn that source into inspected render and atlas output. Read the README for
the public overview; do not treat generated files or this guide as product
source.

## Project Shape

MVG is an Odin-first asset format and renderer for small UI vector assets.
The source of truth is `.mvg`, not SVG and not generated JSON.

Main folders:

- `model/`: core MVG model, lexer, parser, validator, formatter, JSON dump,
  and bounds logic.
- `model/renderer/`: scalar renderer backend, path/stroke/raster/blend code,
  STB PNG writing, and STB truetype text-to-path support.
- `cli/`: thin command wrapper over `model` and `model/renderer`.
- `examples/one-shots/`: isolated component studies with their own theme.
- `examples/material-dark/`: a cohesive Material Dark UI sample, including
  reusable controls.
- `examples/material-light/`: the Material component set rendered with a light
  semantic theme.
- `examples/night-poster/`: the same component contract restyled as a
  hard-edged neo-brutalist event UI.
- `assets/fonts/`: the four IBM Plex fonts required by MVG's bundled examples.
- `showcase/`: curated public renders; ordinary generated output remains in
  ignored `target/` directories.

## Agent Workflow

Use an example directory as a self-contained project. Its `theme.mvg` supplies
colors and fonts to the component files beneath it. Author `.mvg` only:
`*.mvg.json`, PNG renders, atlas images, and layouts are generated diagnostic
or delivery output, never hand-edited source.

For a focused visual iteration:

```sh
make build
target/bin/mvg fmt examples/night-poster --check
target/bin/mvg validate examples/night-poster
target/bin/mvg inspect examples/night-poster
target/bin/mvg render examples/night-poster --asset card --out target/review/card.png
```

Open and assess the rendered PNG after every meaningful visual change. Render
the default asset and each affected variant; a valid file can still have poor
contrast, clipped effects, or unintended geometry. Use `render-all` to review
the full set, then `pack` to confirm atlas output once the component set is
ready:

```sh
target/bin/mvg render-all examples/night-poster --out target/review/night-poster
target/bin/mvg pack examples/night-poster --out target/pack/night-poster --debug-overlay
```

Keep temporary review output under ignored `target/`. Only add a PNG to
`showcase/` when it is deliberately curated for the repository's public
presentation. `--debug-overlay` writes a separate atlas companion showing
canvas, stretch-body, content, and named-slice bounds; it never changes the
runtime atlas.

## Verification Commands

Use these from the repo root:

```sh
make build
make check
make test
make smoke
```

Windows users can run the equivalent root-level `build.bat`, `check.bat`,
`test.bat`, and `smoke.bat` wrappers. They require `odin` on `PATH` and do not
require Make, Bash, or PowerShell.

`make smoke` is the current end-to-end check. It builds the CLI, validates
examples, emits JSON snapshots, inspects the project, renders every example
PNG, packs the examples into atlas/layout output, and checks formatting.

Run `make check` for any code or source edit. Run `make test` and `make smoke`
before handing off parser, renderer, CLI, or example changes. `inspect` can
report visual bounds beyond a fixed source canvas; investigate new warnings and
use a larger canvas plus `body`/`content` metadata when an asset needs
permanent effect space.

## Current CLI

Implemented:

- `mvg parse <file-or-directory> [--json-out <file>]`
- `mvg validate <file-or-directory>`
- `mvg fmt <file-or-directory> [--check] [--out <file>]`
- `mvg inspect <file-or-directory>`
- `mvg render <file-or-directory> [--asset <id>] [--variant <name>] [--out <file.png>]`
- `mvg render-all <file-or-directory> --out <directory>`
- `mvg pack <file-or-directory> --out <directory> [--debug-overlay]`

Not implemented:

- SVG output; SVG import is intentionally out of scope.
- Preview/editor UI.
- Groups, transforms, and richer shared-fragment reuse.

## MVG Source Conventions

Use a shared `theme.mvg` file for colors and fonts. Each example directory is
an independent project, so its theme applies to the assets below it:

```mvg
colors light

palette {
  surface = $slate2
  button_top = $blue9
  button_text = $gray1
}

fonts {
  sans = "assets/fonts/IBMPlexSans-Regular.ttf"
  sans_bold = "assets/fonts/IBMPlexSans-Bold.ttf"
}
```

Product themes should expose semantic palette names:

```mvg
text label "Create" 46 30 size 18 font @sans_bold {
  fill @button_text
}
```

Built-in standard colors are referenced with `$name`, for example `$slate2`,
`$blue9`, or `$redA5`. The `A` scales are alpha colors. Use `colors light` or
`colors dark` to select the built-in table. Project-defined palette colors and
font roles are referenced with `@name`. Bare palette and font references are
invalid source syntax.

Prefer direct `$` standard colors for isolated examples. Define a palette only
when a cohesive project benefits from semantic product tokens, as in
`examples/material-dark/`. Avoid per-component raw colors or direct font paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Skytrias/mvg](https://github.com/Skytrias/mvg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
