---
trigger: always_on
description: *A guide for AI agents and humans working on this repository. It explains what the project is, how it works, where to change code safely, and how to validate changes.*
---

# AGENTS.md - The auto-uv-env Story

*A guide for AI agents and humans working on this repository. It explains what the project is, how it works, where to change code safely, and how to validate changes.*

---

## What This Repo Is

**auto-uv-env** is a shell integration tool that automatically creates, activates, and deactivates Python virtual environments using `uv` when users move between directories.

It is a small system with a strict contract:

1. `auto-uv-env` (the main executable) decides *what should happen* and emits directives.
2. Shell adapters (`bash`, `zsh`, `fish`) decide *how to apply* those directives in the current shell process.

This split keeps shell state mutation out of the main executable and reduces command injection risk.

```
+--------------------------------------------------------------------------------+
|                                auto-uv-env repo                                |
+--------------------------------------------------------------------------------+
|                                                                                |
|  +----------------------+      directives       +---------------------------+  |
|  | auto-uv-env          |---------------------->| shell adapter             |  |
|  | (policy + detection) |                       | (bash/zsh/fish runtime)   |  |
|  +----------------------+                       +-------------+-------------+  |
|                                                           |                    |
|                                                           | source activate    |
|                                                           v                    |
|                                                   +---------------+            |
|                                                   | .venv         |            |
|                                                   | (uv-managed)  |            |
|                                                   +-------+-------+            |
|                                                           |                    |
|                                                           v                    |
|                                                   +---------------+            |
|                                                   | uv + python   |            |
|                                                   +---------------+            |
|                                                                                |
+--------------------------------------------------------------------------------+
```

---

## Architecture Diagrams

### Runtime Activation Flow

```
+-------------------+       +----------------------+       +----------------------+
| shell hook        |------>| auto-uv-env          |------>| directives           |
| (PWD changed)     |       | --check-safe <dir>   |       | CREATE_VENV=1        |
+---------+---------+       +----------+-----------+       | PYTHON_VERSION=3.11  |
          |                            |                   | ACTIVATE=/path/.venv |
          |                            |                   | DEACTIVATE=1         |
          |                            v                   +----------+-----------+
          |                  reads pyproject.toml                     |
          |                  validates venv name                      |
          |                  checks ignore file                       |
          |                                                           v
          |                                            +---------------------------+
          +------------------------------------------->| shell adapter applies     |
                                                       | uv python install         |
                                                       | uv venv                   |
                                                       | source activate           |
                                                       | deactivate                |
                                                       +---------------------------+
```

### Repository Lifecycle

```
+-------------------+    +-------------------+    +------------------------------+
| local change      |--->| test + lint       |--->| release automation           |
| auto-uv-env/*     |    | test/*.sh         |    | scripts/release.sh           |
| share/*           |    | pre-commit        |    | github tag workflow          |
+-------------------+    +-------------------+    +------------------------------+
         |                          |                               |
         |                          |                               |
         v                          v                               v
+-------------------+    +-------------------+    +------------------------------+
| docs source       |    | CI workflows      |    | distribution channels        |
| README.md         |    | .github/workflows |    | GitHub Releases              |
| docs/* (Jekyll)   |    | lint/test/security|    | Homebrew tap                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ashwch/auto-uv-env](https://github.com/ashwch/auto-uv-env) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
