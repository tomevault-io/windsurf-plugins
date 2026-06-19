---
trigger: always_on
description: >-
---


# Profile Shortcut

## Workflow checklist

```
- [ ] Simple shortcut (Mode A) OR extend `my` dispatcher (Mode B)?
- [ ] Mode A: command name + repo name | Mode B: group name + command name + target
- [ ] Resolve target path; verify it exists
- [ ] Detect platform and active shell
- [ ] Read target README → pick underlying command (or ask user)
- [ ] Mode A: add/replace flat function | Mode B: extend registry hashtables OR flat switch + help UI
- [ ] Mode B: detect registry (`$MyCommands`) vs flat switch before editing
- [ ] Reload profile in the same shell
- [ ] Test with a real invocation and flags (not only help / my / my <group>)
- [ ] If wrapping a .ps1 with param(), verify PositionalBinding=$false + ValueFromRemainingArguments
- [ ] Confirm: mode, shell, profile path, README source, usage
```

## Purpose

Help the user create custom shell commands in their **default terminal profile** that jump into a specific repo folder and run a repo-dependent command.

Works on **Windows, macOS, and Linux** using whatever shell the user's terminal session uses (PowerShell, bash, zsh, or fish).

Examples of repo-dependent commands (typically found in README):

```text
dotnet build
npm run build
pnpm dev
python main.py
cargo build
docker compose up
```

## When to use this skill

Use this skill when the user wants to:

* Create a shortcut command in their terminal
* Add a function or alias to their shell profile
* Run a repo-specific command from a specific folder
* Make a command like `bapp`, `build-api`, `run-site`, or another custom name
* Group multiple shortcuts under a personal CLI: `my <group> <command> [args]`
* Reload their shell profile
* Pass extra arguments to the underlying command

## Choose shortcut mode

| Mode | Use when |
|------|----------|
| **A — Simple shortcut** | One repo, one action, rarely grows |
| **B — `my` dispatcher** | Multiple repos/actions; user wants one entry point and grouped help |

**Decision rules:**

1. **Read the profile first** — detect which Mode B shape already exists:
   - `$MyCommands` hashtable → **registry pattern** (extend hashtables; see below)
   - `function my` with `switch` branches → **flat switch pattern** (add help row + switch branch)
   - No `my` → choose skeleton based on expected growth (see Mode B variants)
2. If `function my` already exists → **extend Mode B**; do not add another top-level function for the same intent.
3. User asks for grouped, personal, or namespaced shortcuts → **Mode B**.
4. Single repo, single action, no existing `my` → offer both; default to **Mode A** unless the user prefers grouping.

Mode B is **PowerShell-only** today (colored help + `$args` forwarding). Mode A works on bash, zsh, fish, and PowerShell.

**Mode B variants (PowerShell):**

| Variant | Use when |
|---------|----------|
| **Registry** (recommended) | User already has `$MyCommands`, or expects many groups/commands |
| **Flat switch** | First `my` with only 1–2 commands; minimal boilerplate |

Concept for Mode B:

```text
my <group> <command> [args...]
```

- `my` — generic personal namespace (not user-specific)
- `<group>` — project family or area (e.g. framework, tooling, client work)
- `<command>` — short action (build, dev, migrate, test, …)
- `[args]` — forwarded to the real tool unchanged

User runs `my` alone for full help, `my <group>` for group-scoped help.

## Required first step

Gather fields based on mode. If the user already provided values, do not ask again.

### Mode A — Simple shortcut

1. **Command name** — the shortcut to type (e.g. `build-api`, `dev-site`).
2. **Repo name** — folder name (e.g. `MyApi`, `my-site`), not the full path.

```text
What should the shortcut be called, and which repo is it for?

Example:
Command name: build-api
Repo name: MyApi
```

### Mode B — `my` dispatcher

1. **Group name** — lowercase, short domain label (e.g. `api`, `web`, `tools`).
2. **Command name** — short verb/noun within that group (e.g. `build`, `dev`, `test`).
3. **Target** — repo folder name or full path (resolve like Mode A).

Group by **domain/intent**, not necessarily folder layout — a script inside repo A can live under group B if that matches how the user thinks about it.

If `AskQuestion` is available, use it to gather missing fields in one step (include mode choice when unclear).

## Resolve repo name to path

If the user gives a **full path** instead of a repo name, use it directly.

Otherwise resolve `REPO_NAME` by checking these locations **in order** (first match wins):

| Platform | Search paths |
|----------|--------------|
| Windows | `%USERPROFILE%\GitHub\REPO_NAME`, `%USERPROFILE%\Projects\REPO_NAME`, `%USERPROFILE%\Developer\REPO_NAME`, `%USERPROFILE%\repos\REPO_NAME` |
| macOS / Linux | `~/GitHub/REPO_NAME`, `~/Projects/REPO_NAME`, `~/Developer/REPO_NAME`, `~/repos/REPO_NAME` |

Run a directory check in the shell (e.g. `Test-Path` on Windows, `test -d` on Unix) before proceeding.

If no folder is found, ask the user for the full repo path.

After resolving the path, **verify the folder exists** before reading the README or editing any profile.

**Mode B registry:** add the resolved path to `$MyRepoPaths` under a stable key (usually the folder name). Reference that key from `Entry.Repo` in `$MyCommands`.

## Detect platform and shell


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lasaths/profile-shortcut](https://github.com/lasaths/profile-shortcut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
