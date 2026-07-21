---
trigger: always_on
description: Lavi is a Neovim colorscheme with theme generators for terminal emulators and CLI tools. The palette is defined once in Lua using [lush.nvim](https://github.com/rktjmp/lush.nvim) and compiled into output files for each supported application.
---

# AGENTS.md

Lavi is a Neovim colorscheme with theme generators for terminal emulators and CLI tools. The palette is defined once in Lua using [lush.nvim](https://github.com/rktjmp/lush.nvim) and compiled into output files for each supported application.

## Project Structure

```
lavi/
├── lua/lush_theme/lavi/         # SOURCE - edit these files
│   ├── palette/                 # Canonical color palette (variant-aware)
│   │   ├── init.lua             #   Resolver: picks the active variant and builds it
│   │   ├── build.lua            #   Builder: derives the full palette from seed colors
│   │   └── variants/            #   Per-variant seed colors
│   │       ├── lavi.lua         #     Default variant (the base seed set)
│   │       ├── deep.lua         #     Dark, low-contrast slate neutral treatment
│   │       └── deep-ember.lua   #     deep + sodium-style accents (extends deep.lua)
│   ├── lavi.lua                 # Neovim highlight groups
│   ├── init.lua                 # Entry point (returns lavi.lua)
│   ├── lualine.lua              # Lualine statusline theme
│   ├── transforms.lua           # Shared build helpers (to_json, to_nix, to_toml, to_kdl, compile_palette)
│   ├── alacritty.lua            # Theme generators for external apps
│   ├── bottom.lua               #   Each exports: colors, transform, [transform_nix]
│   ├── btop.lua
│   ├── clipse.lua
│   ├── dank-material-shell.lua
│   ├── foot.lua
│   ├── ghostty.lua
│   ├── kitty.lua
│   ├── opencode.lua
│   ├── pi.lua
│   ├── textmate.lua
│   ├── wezterm.lua
│   ├── zellij.lua
│   └── base16.lua
│
├── lua/lavi/                    # SOURCE - runtime support files
│   ├── init.lua                 # Module annotations
│   ├── setup.lua                # Colorscheme setup (applies config + highlight groups)
│   └── types.lua                # Type definitions (lavi.Config)
│
│                                # NOTE: the default "lavi" variant writes the
│                                # unsuffixed paths below; other variants write
│                                # "-<variant>" siblings (e.g. colors/lavi-deep.lua,
│                                # contrib/ghostty/lavi-deep-ember.conf).
├── colors/lavi.lua              # GENERATED - compiled neovim colorscheme
├── colors/lavi_dev.lua          # SOURCE - development colorscheme loader (uses lush at runtime)
├── lua/lavi/palette.lua         # GENERATED - compiled palette (plain hex strings, no lush)
├── lua/lualine/themes/lavi.lua  # GENERATED - compiled lualine theme
├── contrib/                     # GENERATED - theme files + READMEs for each app
│   ├── alacritty/
│   │   ├── lavi.toml
│   │   └── README.md            # Generated from docs metadata in build.lua
│   ├── base16/lavi.yaml
│   ├── bottom/
│   │   ├── lavi.toml
│   │   └── README.md
│   ├── btop/
│   │   ├── lavi.theme
│   │   └── README.md
│   ├── clipse/
│   │   ├── lavi.json
│   │   └── README.md
│   ├── dank-material-shell/
│   │   ├── lavi.json
│   │   └── README.md
│   ├── foot/
│   │   ├── lavi.ini
│   │   └── README.md
│   ├── ghostty/
│   │   ├── lavi.conf
│   │   └── README.md
│   ├── kitty/
│   │   ├── lavi.conf
│   │   └── README.md
│   ├── opencode/
│   │   ├── lavi.json
│   │   └── README.md
│   ├── textmate/
│   │   ├── lavi.tmTheme
│   │   └── README.md
│   ├── wezterm/
│   │   ├── lavi.toml
│   │   └── README.md
│   ├── windows_terminal/
│   │   ├── lavi.json
│   │   └── README.md
│   └── zellij/
│       ├── lavi.kdl
│       └── README.md
├── nix/themes/                  # GENERATED - nix expressions for home-manager
│   ├── alacritty.nix
│   ├── bottom.nix
│   ├── clipse.nix
│   ├── foot.nix
│   └── ghostty.nix
│
├── build.lua                    # Build orchestrator
├── justfile                     # Build commands
├── flake.nix                    # Nix flake (packages, home-manager module, dev shell)
└── nix/homeManagerModules/
    └── default.nix              # Home-manager module with per-app enable options
```

## Source vs Generated Files

Source files live in two locations:

- `lua/lush_theme/lavi/` - palette, highlight groups, and theme generators
- `lua/lavi/` - runtime support (setup, types, module annotations)
- `colors/lavi_dev.lua` - development colorscheme loader

Generated files (do not edit directly):

- `colors/lavi.lua` - compiled neovim colorscheme
- `lua/lavi/palette.lua` - compiled palette (plain hex strings, no lush)
- `lua/lualine/themes/` - compiled lualine theme
- `contrib/` - theme files and READMEs for external apps
- `nix/themes/` - nix expressions

Every generated artifact is produced once per variant (see [Variants](#variants)).
The default `lavi` variant keeps the unsuffixed paths above; other variants get
`-<variant>` siblings (`colors/lavi-deep.lua`, `contrib/ghostty/lavi-deep-ember.conf`,
`nix/themes/alacritty-deep.nix`, `lua/lavi/palette-deep.lua`, ...).

CI will auto-regenerate and commit these on pushes to main. PRs will fail if generated files are stale.

## Build System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [b0o/lavi](https://github.com/b0o/lavi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
