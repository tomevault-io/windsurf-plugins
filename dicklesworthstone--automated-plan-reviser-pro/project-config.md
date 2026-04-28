---
trigger: always_on
description: You may NOT delete any file or directory unless I explicitly give the exact command **in this session**.
---

# AGENTS.md — Automated Plan Reviser Pro (APR)

## RULE 1 – ABSOLUTE (DO NOT EVER VIOLATE THIS)

You may NOT delete any file or directory unless I explicitly give the exact command **in this session**.

- This includes files you just created (tests, tmp files, scripts, etc.).
- You do not get to decide that something is "safe" to remove.
- If you think something should be removed, stop and ask. You must receive clear written approval **before** any deletion command is even proposed.

Treat "never delete files without permission" as a hard invariant.

---

## IRREVERSIBLE GIT & FILESYSTEM ACTIONS

Absolutely forbidden unless I give the **exact command and explicit approval** in the same message:

- `git reset --hard`
- `git clean -fd`
- `rm -rf`
- Any command that can delete or overwrite code/data

Rules:

1. If you are not 100% sure what a command will delete, do not propose or run it. Ask first.
2. Prefer safe tools: `git status`, `git diff`, `git stash`, copying to backups, etc.
3. After approval, restate the command verbatim, list what it will affect, and wait for confirmation.
4. When a destructive command is run, record in your response:
   - The exact user text authorizing it
   - The command run
   - When you ran it

If that audit trail is missing, then you must act as if the operation never happened.

---

## Project Overview

**APR (Automated Plan Reviser Pro)** is a CLI tool that automates iterative specification refinement using GPT Pro Extended Reasoning via Oracle browser automation.

### Core Concept

Like numerical optimization converging on a steady state, specification design improves through multiple iterations:

1. **Early rounds** fix major issues (security gaps, architectural flaws)
2. **Middle rounds** refine architecture and interfaces
3. **Later rounds** polish abstractions and fine-tune details

APR automates the tedious parts:
- Bundling documents (README, spec, implementation)
- Sending to GPT Pro 5.2 with Extended Reasoning
- Session management and monitoring
- Round tracking and history

---

## Bash Script Discipline

This is a **pure Bash project** (no embedded languages).

### Bash Rules

- Target **Bash 4.0+** compatibility. Use `#!/usr/bin/env bash` shebang.
- Use `set -euo pipefail` for strict error handling.
- Use ShellCheck to lint all scripts. Address all warnings at severity `warning` or higher.
- Prefer functions over inline code for reusability.
- Use meaningful variable names, avoid single letters except in loops.

### Key Patterns

- **Stream separation** — stderr for human-readable output (progress, errors), stdout for structured data.
- **XDG compliance** — Data in `~/.local/share/apr/`, cache in `~/.cache/apr/`.
- **No global `cd`** — Use absolute paths; change directory only when necessary.
- **Graceful degradation** — gum → ANSI colors, Oracle global → npx.

---

## Project Architecture

```
automated_plan_reviser_pro/
├── apr                  # Main script (~800 LOC)
├── install.sh           # Curl-bash installer
├── README.md            # Comprehensive documentation
├── AGENTS.md            # This file
├── VERSION              # Semver version file
├── LICENSE              # MIT License
├── workflows/           # Example workflow configs
│   └── fcp-example.yaml
├── templates/           # Prompt templates
│   └── standard.md
└── lib/                 # Shared functions (future)
```

### Per-Project Configuration

When APR is used in a project, it creates:

```
<project>/
└── .apr/
    ├── config.yaml           # Default workflow setting
    ├── workflows/            # Workflow definitions
    │   └── <name>.yaml
    ├── rounds/               # GPT Pro outputs
    │   └── <workflow>/
    │       └── round_N.md
    └── templates/            # Custom prompt templates
```

---

## Workflow Configuration Format

```yaml
# .apr/workflows/example.yaml

name: example
description: Example workflow for project specification

documents:
  readme: README.md
  spec: SPECIFICATION.md
  implementation: docs/implementation.md  # Optional

oracle:
  model: "5.2 Thinking"

rounds:
  output_dir: .apr/rounds/example
  impl_every_n: 4  # Auto-include implementation every 4th round

template: |
  First, read this README:
  ...
```

**Key config options:**

- `impl_every_n: N` — Automatically include implementation document every Nth round (e.g., rounds 4, 8, 12...)
- This keeps the spec grounded in implementation reality without manual `--include-impl` flags

**Run APR from your project directory** where your README, spec, and implementation files live. The `.apr/` configuration is per-project.

---

## Code Editing Discipline

- Do **not** run scripts that bulk-modify code (codemods, invented one-off scripts, giant `sed`/regex refactors).
- Large mechanical changes: break into smaller, explicit edits and review diffs.
- Subtle/complex changes: edit by hand, file-by-file, with careful reasoning.

---

## Backwards Compatibility & File Sprawl

We optimize for a clean architecture now, not backwards compatibility.

- No "compat shims" or "v2" file clones.
- When changing behavior, migrate callers and remove old code.
- New files are only for genuinely new domains that don't fit existing modules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/automated_plan_reviser_pro](https://github.com/Dicklesworthstone/automated_plan_reviser_pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
