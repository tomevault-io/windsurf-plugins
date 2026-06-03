---
trigger: always_on
description: 1. **Public Repository Warning**: This is a PUBLIC repository. Never include internal/sensitive Atlassian information in the project.
---

# Atlaspack Agent Guide

## Critical Rules ⚠️

1. **Public Repository Warning**: This is a PUBLIC repository. Never include internal/sensitive Atlassian information in the project.

2. **File Operations**:
   - Always read files before editing them
   - Always prefer editing existing files instead of creating new ones
   - Always use specialized tools (read_files, edit, write) instead of bash commands where possible
   - Never create documentation files unless directed

3. **Communication Style**:
   - Always be concise and technical
   - Always output text directly to user, NOT via bash echo or comments
   - Always request more information when necessary
   - Never use emoji

4. **Code Quality**:
   - Always run relevant tests for the area of code being modified
   - Always improve the test suite when fixing issues instead of just reading the code or using CLI
   - Always format files after editing using the relevant tool
   - Never use placeholders in code, always use real values or ask for them

5. **Git Safety**:
   - Never push or commit unless directed
   - Never update git config
   - Never force push to main/master
   - Never use interactive git commands (`-i` flag)
   - Never skip hooks (--no-verify, --no-gpg-sign)
   - Never run destructive git commands unless directed
   - Never create PRs unless directed

6. **Task Management**:
   - Always use the todo/task tool
   - Always mark todos/tasks as complete immediately after finishing them
   - Never work on multiple todos/tasks simultaneously

7. **When stuck**:
   - Check "Development Workflow Guide" section
   - Ask the developer for clarification
   - Review recent commits for context

## Project Overview

Atlaspack is a high-performance frontend bundler designed to build exceptionally large applications at Atlassian scale. It is written in JavaScript/TypeScript and Rust, forked from Parcel, and optimized for internal Atlassian product development. While publicly available, it is not intended for production use outside Atlassian.

### Core Architecture

Atlaspack follows a plugin-based architecture with these key components:

1. **Core Engine** - Orchestrates the build process through request tracking and caching
2. **Asset Graph** - Dependency graph that tracks all assets and their relationships
3. **Bundle Graph** - Determines how assets are grouped into output bundles
4. **Plugin System** - Specialised and modular extensions
   - **Transformers**: Convert source files to Atlaspack-compatible format
   - **Resolvers**: Find dependencies and resolve import paths
   - **Bundlers**: Determine how assets are grouped into bundles
   - **Namers**: Generate output filenames
   - **Packagers**: Concatenate assets into final bundle files
   - **Optimizers**: Minify and optimize bundled code
   - **Reporters**: Report build progress and results
   - **Compressors**: Compress output files

### Project Structure

- Atlaspack uses the `yarn` package manager for JS/TS packages
- Atlaspack uses Lerna for managing its multiple JS/TS packages
- Atlaspack uses a Cargo workspace for Rust crates (defined in root `Cargo.toml`)

```
packages/                                # JavaScript/TypeScript packages
├── core/                                # Core Atlaspack packages
├── transformers/                        # Transformer plugins (JS, CSS, HTML, etc.)
├── bundlers/                            # Bundler plugins
├── optimizers/                          # Optimizer plugins (minifiers, etc.)
├── packagers/                           # Packager plugins
├── resolvers/                           # Module resolvers
├── namers/                              # Bundle naming strategies
├── reporters/                           # Build reporters (CLI, dev server, etc.)
├── runtimes/                            # Runtime code injected into bundles
├── utils/                               # Shared utilities
├── dev/                                 # Development tools
│   ├── atlaspack-inspector/             # Build inspector UI
│   ├── query/                           # Query tool for builds
│   └── bundle-stats-cli/                # Bundle statistics
└── examples/                            # Example projects for testing
crates/                                  # Rust crates (Cargo workspace)
├── atlaspack/                           # Main Atlaspack crate
├── atlaspack_core/                      # Core types and asset graph
├── atlaspack_config/                    # Configuration handling
├── atlaspack_filesystem/                # FS operations
├── atlaspack_sourcemap/                 # Source map handling
├── atlaspack_monitoring/                # Sentry integration for crash reporting
├── atlaspack_plugin_transformer_js/     # SWC-based JS transformer
├── atlaspack_plugin_transformer_css/    # Lightning CSS transformer
├── atlaspack_plugin_transformer_html/   # HTML parser/transformer
├── atlaspack_plugin_transformer_image/  # Image optimization
├── atlaspack_plugin_resolver/           # Module resolver
├── atlaspack_plugin_rpc/                # Plugin RPC communication
├── atlaspack_swc_runner/                # SWC runner utilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atlassian-labs/atlaspack](https://github.com/atlassian-labs/atlaspack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
