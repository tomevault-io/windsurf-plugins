---
trigger: always_on
description: - Always answer the user in Japanese.
---

# AGENTS.md

## Global Communication Rules

- Always answer the user in Japanese.
- Instructions intended for the AI (including this document and the return
  values of `Tools` methods) must remain in English, even though user-facing
  replies are in Japanese.

## Repository Structure

This repository contains Open WebUI extensions, with some components managed as Git submodules.

### Submodules

- `graphiti/` - Graphiti memory system extensions (separate repository)
- `references/open-webui/` - Reference submodule

### Working With Open WebUI Core

1. The upstream Open WebUI core lives in the `references` submodule. Before
   consulting or copying code, update the submodule to the latest commit so you
   rely on current behavior.
2. Implement features only after inspecting the actual core and related
   libraries; never guess about undocumented behavior.
3. When adapting snippets, keep them consistent with the upstream style and
   ensure any dependencies are mirrored locally.
4. Before implementing a new Filter/Tool/Action, inspect at least one existing
   implementation of the same type in this repository (and in
   `references/open-webui` if needed) so you follow established patterns and
   edge-case handling.

## Commit Message Rules

### Important: Check Submodule Commit Style First

Before committing to a submodule, **always check the existing commit history** using:

```bash
git log -5 --format=medium
```

Each submodule may have its own commit message style. Follow the existing pattern.

### Graphiti Submodule Style

Uses **Conventional Commits** format:

```text
type(scope): short description

Optional detailed description explaining what and why.
- Bullet points for multiple changes
- Keep each point concise
```

**Types:**

- `feat` - New feature
- `fix` - Bug fix
- `chore` - Maintenance (version updates, dependencies, etc.)
- `docs` - Documentation only
- `refactor` - Code refactoring without feature changes
- `test` - Adding or updating tests

**Scopes:**

- `filter` - Filter components
- `tools` - Tools components
- `pipeline` - Pipeline components
- `action` - Action components

**Examples:**

```text
feat(tools): add get_recent_episodes method for chronological episode retrieval

- Add get_recent_episodes() method with limit/offset pagination
- Sort episodes by created_at in descending order (newest first)
- Optimize database queries to fetch only offset+limit episodes
- Update version to 0.3
```

```text
fix(filter): Fix null pointer exception in D3.js graph tick handler

Add null checks before calling getBBox() on text nodes in the force
simulation tick event.
```

### Root Repository Style

Uses **Conventional Commits** format with English descriptions.

## Development Guidelines

### General Principles

1. **Version Management**
   - Update version numbers when adding features or fixing bugs
   - Follow semantic versioning principles

2. **Before Committing**
   - Verify changes work correctly
   - Check for linting errors
   - Review the commit message style of the target repository/submodule

### Valves vs UserValves

Applies to Tools, Filters, and Pipes.

- Follow the official pattern: declare both classes as nested subclasses of the
  Filter/Tools/Pipe you are implementing, inherit from `BaseModel`, and always
  call `self.valves = self.Valves()` inside `__init__` for admin-controlled
  defaults.
- `Valves` describe instance-wide knobs (API keys, DB backends, semaphore limits,
  etc.) and may include fields like `priority` for filter ordering. `UserValves`
  expose only per-user preferences (feature toggles, language choices, etc.).
- Fetch user valves via
  `self.UserValves.model_validate((__user__ or {}).get("valves", {}))` so that
  raw dict payloads from Open WebUI are validated, missing fields fall
  back to defaults, and you avoid sharing mutable state.
- Treat the object stored under `__user__["valves"]` as a Pydantic model, not a
  dict; access attributes (`__user__["valves"].test_user_valve`) or cast with
  `dict(__user__["valves"])` if you truly need a mapping. Indexing into it as if
  it were a dict returns defaults and ignores user-set values.
- Keep the schemas disjoint and document each `Field` carefully so the UI can
  generate appropriate controls (e.g., `Literal[...]` for dropdowns, `bool` for
  switches). Always include `pass` at the end of each class as recommended in
  the core documentation.
- The current UI does not expose true multi-select widgets, so when you need
  multiple choices, accept comma-separated strings (e.g., `"choiceA,choiceB"`)
  and document that input format for users.

### Tool Implementation

#### General Guidelines

- Assume every public method you add will be called by the AI. Provide precise
  docstrings that describe the purpose, parameters, expected argument formats,
  possible errors, and example correction strategies.
- Tools are never invoked directly by humans; the AI alone decides when to
  call them and assembles the arguments, so design signatures and validation
  with that autonomous caller in mind.
- Expose only callable methods; keep helpers outside of the `Tools` class so the
  AI cannot invoke unsupported operations (method names starting with `_` are
  not an exception).

#### CRITICAL: No Helper Methods in Tools Class


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Skyzi000/open-webui-extensions](https://github.com/Skyzi000/open-webui-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
