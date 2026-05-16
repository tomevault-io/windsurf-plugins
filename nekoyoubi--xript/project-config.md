---
trigger: always_on
description: xript is a platform specification for making any application moddable. Users write JavaScript; xript standardizes the bindings, capability model, sandboxing guarantees, documentation, and tooling. See [spec/vision.md](spec/vision.md) for the full vision.
---

# CLAUDE.md

## What is xript?

xript is a platform specification for making any application moddable. Users write JavaScript; xript standardizes the bindings, capability model, sandboxing guarantees, documentation, and tooling. See [spec/vision.md](spec/vision.md) for the full vision.

## Repository Structure

This is a monorepo managed via npm workspaces.

```
xript/
├── spec/           # the specification (manifest schema, capability model, etc.)
├── runtimes/       # language-specific runtime implementations
│   ├── js/         # universal runtime (@xriptjs/runtime, QuickJS WASM sandbox)
│   ├── node/       # Node.js-optimized runtime (@xriptjs/runtime-node, vm-based)
│   ├── rust/       # native Rust runtime (xript-runtime, QuickJS via rquickjs)
│   └── csharp/     # C# runtime (Xript.Runtime, Jint sandbox)
├── renderers/      # fragment renderer implementations
│   └── ratatui/    # terminal renderer (xript-ratatui, Ratatui widgets)
├── tools/          # ecosystem tooling (validator, typegen, docgen, init, sanitize, wiz)
│   └── wiz/        # interactive TUI wizard (xript-wiz, powered by xript fragments)
├── docs/           # documentation site (Astro + Starlight), deployed to xript.dev
└── examples/       # example manifests and integrations
```

## Tech Stack

- **Docs site**: Astro with Starlight, deployed to GitHub Pages via GitHub Actions (live demos depend on `@xriptjs/runtime` -- CI builds sanitize then runtime before docs)
- **Package management**: npm workspaces
- **Language**: TypeScript throughout
- **Runtime sandbox (js)**: QuickJS compiled to WASM via `quickjs-emscripten`, runs in browser, Node, Deno, and more
- **Runtime sandbox (node)**: Node.js `vm` module with `codeGeneration: { strings: false, wasm: false }`
- **Runtime sandbox (rust)**: QuickJS via `rquickjs` (native), for Rust host applications
- **Runtime sandbox (csharp)**: Jint (pure C# JS interpreter), for .NET host applications
- **Fragment renderer (ratatui)**: `xript-ratatui` crate renders `application/x-ratatui+json` fragments as native Ratatui terminal widgets
- **Test runner**: Node.js built-in test runner (`node --test`), xUnit for C#

## Development Commands

```sh
npm install                            # install all workspace dependencies
npm run docs:dev                       # run the docs site locally on port 4351
npm run docs:build                     # build the docs site for production

# build and test individual packages (build sanitize first — runtimes depend on it)
npm run build --workspace=tools/sanitize           # build the HTML sanitizer
npm test --workspace=tools/sanitize                # run sanitizer tests (71 tests)
npm run build --workspace=runtimes/js              # build the universal runtime
npm test --workspace=runtimes/js                   # run universal runtime tests (97 tests)
npm run build --workspace=runtimes/node            # build the Node.js runtime
npm test --workspace=runtimes/node                 # run Node.js runtime tests (97 tests)
npm run build --workspace=tools/validate            # build the validator
npm test --workspace=tools/validate                 # run validator tests (25 tests)
npm run build --workspace=tools/typegen            # build the type generator
npm test --workspace=tools/typegen                 # run typegen tests (31 tests)
npm run build --workspace=tools/docgen             # build the doc generator
npm test --workspace=tools/docgen                  # run docgen tests (28 tests)
npm run build --workspace=tools/init               # build the init CLI
npm test --workspace=tools/init                    # run init tests (34 tests)
npm run build --workspace=tools/cli                # build the unified CLI
npm test --workspace=tools/cli                     # run CLI tests (29 tests)

# build and test Rust packages
cd runtimes/rust && cargo build                    # build the Rust runtime
cd runtimes/rust && cargo test                     # run Rust runtime tests (45 tests)
cd renderers/ratatui && cargo build                # build the Ratatui fragment renderer
cd renderers/ratatui && cargo test                 # run Ratatui renderer tests (58 tests)
cd tools/wiz && cargo build                        # build the TUI wizard
cd tools/wiz && cargo test                         # run TUI wizard tests (35 tests)

# build and test the C# runtime
dotnet build runtimes/csharp/Xript.Runtime.sln     # build the C# runtime
dotnet test runtimes/csharp/Xript.Runtime.sln      # run C# runtime tests (116 tests)

# unified CLI (run from repo root after npm install)
npx xript validate <manifest.json>     # validate a manifest against the spec schema
npx xript typegen <manifest.json>      # generate TypeScript definitions (stdout)
npx xript typegen <m.json> -o out.d.ts # generate TypeScript definitions (file)
npx xript docgen <m.json> -o docs/     # generate markdown documentation
npx xript init                         # scaffold a new xript project (interactive)
npx xript init --yes                   # scaffold with defaults (no prompts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nekoyoubi/xript](https://github.com/nekoyoubi/xript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
