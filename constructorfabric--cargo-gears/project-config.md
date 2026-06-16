---
trigger: always_on
description: cli reference to help with the development of constructor fabric gears framework. It helps with the development of
---


# General Guidelines to follow

- When adding new dependencies use `cargo add`, do not edit Cargo.toml manually
- When linting, use lint command of the cli to check if there are any lint errors.
  Do not try to run cargo check, clippy or fmt by your own.
- Always verify that the application runs successfully after modifying the code.
- Unless the user specifically mentions to use a custom module, prefer to use system modules
  instead of implementing your own ones. Pay attention to the "deps" section of the modules, as the module will be
  required to be added in order to be used.
- Do not try to create a module from scratch, always use the generate module command to create a new module.

## Invocation Forms

The crate exposes a single entrypoint:

- **[`cargo gears`]** Cargo subcommand form via the `cargo-gears` binary

Example:

```bash
cargo gears generate workspace /tmp/my-app
```

## Objective

This CLI is a tool for automating gears development, a Rust framework. You can get more information about it in:

- Gears repository main: https://github.com/constructorfabric/cyberware-rust
- Gears libraries(the ones that leverage this CLI tool) are located
  in https://github.com/constructorfabric/cyberware-rust/tree/main/libs
- More documentation of the project will be located in https://github.com/constructorfabric/cyberware-rust/tree/main/docs

Clone(shallow) the repo to .gears folder (create it if it doesn't exist), and use it as a reference.
If so, prefer to use the ssh version instead of https to avoid authentication issues.

## Command Tree

```text
cargo gears
├── generate
│   ├── workspace
│   ├── module
│   └── config
├── new
├── config
│   ├── mod
│   │   ├── add
│   │   ├── db
│   │   │   ├── add
│   │   │   ├── edit
│   │   │   └── rm
│   │   └── rm
│   └── db
│       ├── add
│       ├── edit
│       └── rm
├── src
├── help
│   ├── schema
│   ├── src
│   └── topic
├── lint
├── ls
│   └── modules
├── manifest
│   ├── validate
│   └── ls
├── test
├── tools
├── run
├── build
└── deploy
```

## Shared Argument Patterns

- **[`-p, --path <PATH>`]** Optional workspace path. When provided to `config ...`, `build`, `run`, `deploy`, `lint`,
  and `test`, relative config paths, manifest paths, generated project locations, and workspace-scoped lint/test resolution use
  that directory as the workspace root. When omitted, the current working directory is used as the workspace root.
- **[`-c, --config <PATH>`]** Config file path. This is required for `config ...` and `deploy` commands because there is
  no default. `build` and `run` no longer accept `--config`; they compose their generation inputs from `Gears.toml`
  and forward the manifest-declared runtime config path through the `GEARS_CONFIG` environment variable.
- **[`--manifest <PATH>`]** Gears manifest path, defaulting to `Gears.toml`, for `manifest`, `build`, `run`,
  `lint`, and `test`.
  For `manifest`, you can combine this with `-p/--path` to resolve relative manifest paths from a selected workspace.
- **[`--app <APP> --env <ENV>`]** Selects a manifest app/environment for manifest-driven `build`, `run`, `lint`, and `test`.
  When omitted, inferred from the manifest: a single app is used automatically; with multiple apps the command
  fails listing available names. For environments, `dev` is selected by default if it exists; otherwise the
  command fails listing available names.
- **[`--name <NAME>`]** For `build` and `run`, overrides the generated server project and binary name that would
  otherwise default to the config filename stem.
- **[`-v, --verbose`]** Usually enables more logging or richer output.
- **[name validation]** Config-managed names for modules, DB servers, and generated server names only allow letters,
  numbers, `-`, and `_`.

## What the Tool Manages

From the current implementation, the CLI is mainly for:

- **[workspace scaffolding]** Initialize a Gears workspace and add module templates
- **[config management]** Enable modules and patch YAML config sections
- **[server generation]** Generate a runnable Cargo project under the manifest `workspace.generated-dir` directory
  (default `.gears/<name>/`)
- **[manifest orchestration]** Read `Gears.toml` to separate generation metadata from runtime YAML config
- **[build/run/deploy]** Build, run, or package that generated server as a Docker image
- **[source inspection]** Resolve Rust source for crates/items through workspace metadata or crates.io
- **[module inspection]** List workspace-discovered and system-registry modules
- **[tool bootstrap]** Install or upgrade `rustup`, `cargofmt`, and `clippy`

## Top-Level Commands

### `generate`

Generate workspace, module, and config scaffolding from built-in templates or explicit local/Git template sources.

#### `generate workspace`

Synopsis:

```bash
cargo gears generate workspace <path> [--template <TEMPLATE>] [--verbose] [--name <NAME>] [--local-path <PATH>] [--git <URL>] [--subfolder <NAME>] [--branch <NAME>] [--override]
```

Arguments:

- **[`<path>`]** Target directory to initialize
- **[`-t, --template <TEMPLATE>`]** Workspace template name, defaults to `default`
- **[`-v, --verbose`]** Verbose output from `cargo-generate`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [constructorfabric/cargo-gears](https://github.com/constructorfabric/cargo-gears) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
