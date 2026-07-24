---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Ultimate Turbo Modal (UTMR) v3 is a full-featured modal and drawer implementation for Rails applications using Turbo, Stimulus, and Hotwire. It consists of a **Ruby gem** (server-side HTML generation via Phlex) and an **npm package** (client-side behavior via Stimulus). v3 is built on the native HTML `<dialog>` element, eliminating external dependencies for focus trapping and transitions.

## Project Structure

```
/                               # Ruby gem root
├── VERSION                     # Single source of truth for version (read by both gem and npm)
├── ultimate_turbo_modal.gemspec
├── Gemfile                     # Gem dev dependencies (standard, standard-rails)
├── Rakefile                    # Default task: standard (Ruby linter)
├── CHANGELOG.md
├── lib/
│   ├── ultimate_turbo_modal.rb         # Entry point, factory method, flavor loading
│   ├── ultimate_turbo_modal/
│   │   ├── version.rb                  # Reads VERSION file
│   │   ├── configuration.rb            # Config class + UltimateTurboModal.configure
│   │   ├── base.rb                     # Core Phlex component (HTML rendering)
│   │   ├── railtie.rb                  # Rails integration (hooks helpers into AC/AV)
│   │   └── helpers/
│   │       ├── controller_helper.rb    # inside_modal? method
│   │       ├── view_helper.rb          # modal() and drawer() view helpers
│   │       └── stream_helper.rb        # turbo_stream.modal(:close) helper
│   ├── phlex/
│   │   └── deferred_render_with_main_content.rb  # Phlex mixin for deferred rendering
│   └── generators/ultimate_turbo_modal/
│       ├── base.rb                     # Shared generator logic (JS package detection)
│       ├── install_generator.rb        # `rails g ultimate_turbo_modal:install`
│       ├── update_generator.rb         # `rails g ultimate_turbo_modal:update`
│       └── templates/
│           ├── ultimate_turbo_modal.rb # Initializer template
│           └── flavors/
│               ├── tailwind.rb         # Tailwind v4+ classes (modal + drawer)
│               ├── vanilla.rb          # Vanilla CSS classes (modal + drawer)
│               └── custom.rb           # Empty template for user-defined styling
├── javascript/                 # npm package source
│   ├── package.json            # npm: "ultimate_turbo_modal"
│   ├── index.js                # Entry: Turbo stream actions, frame handlers, exports
│   ├── modal_controller.js     # Stimulus controller (modal + drawer behavior)
│   ├── rollup.config.js        # Build config (ESM output, terser, version replacement)
│   ├── styles/
│   │   └── vanilla.css         # Vanilla CSS flavor styles + animations
│   ├── scripts/
│   │   ├── release-npm.sh      # npm publish script
│   │   └── update-version.js   # Syncs VERSION → package.json version
│   └── dist/                   # Built output (committed, published to npm)
├── script/
│   └── build_and_release.sh    # Combined gem + npm release script
└── demo-app/                   # Rails 8 app for manual testing
    ├── Procfile.dev             # Run with overmind/foreman
    └── ...                     # Uses path gem + link:../javascript for local dev
```

## Architecture

### How the Pieces Fit Together

1. **Server-Side (Ruby Gem)**: The `modal()` or `drawer()` view helper instantiates `UltimateTurboModal.new(...)`, which resolves the configured flavor class (e.g., `UltimateTurboModal::Flavors::Tailwind`). This class inherits from `UltimateTurboModal::Base` (a Phlex component) and defines CSS class constants. The base class renders a `<dialog>` element with data attributes that configure the Stimulus controller, plus inline `<style>` for animations.

2. **Client-Side (npm Package)**: The Stimulus `modal` controller connects when the `<dialog>` appears in the DOM. It calls `showModal()`, manages CSS animations for enter/leave, handles scroll locking (via native dialog), browser history, and dismissal via ESC/click-outside/form-submission. No external animation or focus-trap libraries are needed.

3. **Communication**: Turbo Frames (`<turbo-frame id="modal">`) carry modal content. Turbo Streams can send `modal` stream actions to close modals from the server. Idiomorph is used for intelligent DOM morphing to prevent flicker when modal content updates.

### Flavor System

Flavors are Ruby classes that inherit from `UltimateTurboModal::Base` and define constants for CSS classes. They live in `config/initializers/` in the consuming Rails app (copied there by the install generator). Available flavors:

- **`tailwind`** — Tailwind CSS v4+ (default). Uses utility classes and `group-data-[*]` selectors.
- **`vanilla`** — Plain CSS with classes defined in `javascript/styles/vanilla.css`.
- **`custom`** — Empty template for users to define their own classes.

Each flavor defines **modal constants** and **drawer constants**:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cmer/ultimate_turbo_modal](https://github.com/cmer/ultimate_turbo_modal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
