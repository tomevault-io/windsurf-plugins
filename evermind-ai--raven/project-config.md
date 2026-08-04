---
trigger: always_on
description: Raven AI-collaboration spec. **Read this file before making any code change in this repo.**
---

# AGENTS.md

Raven AI-collaboration spec. **Read this file before making any code change in this repo.**

Scope: Codex / Claude Code / Claude API / any AI-assisted work. When a rule here conflicts with an ad-hoc instruction in conversation, **this file wins** — unless the user *explicitly* says "ignore rule X in AGENTS.md".

Hard constraints only (violations get reverted / rejected). Soft suggestions and style preferences belong in personal notes or conversation, not here. See the [Maintenance](#maintenance) note before adding sections — shorter is better.

| # | Section | Gist |
|---|---|---|
| 1 | [Code comments](#1-code-comments) | Don't comment unless necessary; comments in English |
| 2 | [Branch naming](#2-branch-naming) | `<type>/<snake_desc>`; confirm base before cutting |
| 3 | [Commits](#3-commits-conventional-commits) | Conventional Commits, all-English, `Co-authored-by` trailer |
| 4 | [Dependencies](#4-dependencies-uv-only) | `uv` only — never `pip` / hand-edit lockfile |
| 5 | [Tests](#5-tests) | `uv run pytest`; strict file-naming |
| 6 | [Domain terms](#6-domain-terms) | Consult `CONTEXT-MAP.md` before naming; use canonical terms |
| 7 | [Repository assets](#7-repository-assets) | No report assets, web artifacts, or large files in PRs |

---

## 1. Code comments

### §1.1 Top rule: don't add comments unless necessary

- Match the style of surrounding lines. If neighboring code has no comments, **don't** add one to your new line.
- Comment **only** when:
  - the logic is non-obvious;
  - there's a hidden constraint (e.g. call-order sensitivity, a caller must do X first);
  - you need to explain **why**, not **what** (the name already says what).
- **Don't** add comments that:
  - describe what the code does (`# Increment counter` next to `counter += 1`);
  - mark edits (`# ← new` / `# changed this line`);
  - reference a PR / Issue / locally-visible-only doc path (`# Refs: ...` — invisible to others);
  - describe transient task context (`# For the X bug` — stale once the task is done).

### §1.2 When a comment is required, write it in English

- Repo source comments **must not be in another language** — keep comment language consistent across the repo.

### §1.3 Examples

❌ Non-English review comment copied straight into source:

```python
self.logger = logger.bind(channel=self.name)   # ← new
```

❌ Neighbors have no comments, yet the new line adds a meaningless one:

```python
def __init__(self, config: Any, bus: MessageBus):
    self.config = config
    self.bus = bus
    self._running = False
    self.logger = logger.bind(channel=self.name)   # ← drop this comment
```

✅ Clean, no comment, consistent:

```python
def __init__(self, config: Any, bus: MessageBus):
    self.config = config
    self.bus = bus
    self._running = False
    self.logger = logger.bind(channel=self.name)
```

✅ Rare case that genuinely needs a *why*, in English:

```python
# Bind channel name into logger context so every log entry auto-tags channel.
self.logger = logger.bind(channel=self.name)
```

---

## 2. Branch naming

### §2.1 Format

`<type>/<short-desc>`

| type | Use |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Refactor (not a feature, not a bug fix) |
| `perf` | Performance |
| `chore` | Misc (deps bump, doc structure, etc.) |
| `docs` | Docs only |
| `test` | Tests only |

`short-desc`: **snake_case**, English, 3–5 words describing the change.

| ✅ Good | ❌ Bad |
|---|---|
| `feat/whatsapp_lid_mapping` | `feat/优化` |
| `fix/cron_dst_transition` | `bugfix` |
| `refactor/cli_cron_sentinel` | `huangjie-test` |
| `chore/upgrade_uv` | `tmp` |

### §2.2 Confirm the base before cutting

- Before cutting any branch (`fix` / `feat` / `refactor` / anything), **ask the user which base to cut from** — don't pick one silently.
- If unspecified, default to **`main`** (the integration branch).
- Flow: `git fetch origin main`, then cut from the latest tip.
- Combined with the branch-first rule: **confirm base + cut the branch, then start editing** — never write on a working branch and carve the branch out afterwards.

---

## 3. Commits (Conventional Commits)

### §3.1 Message format

```
<type>(<scope>): <subject>

<body — optional>

<footer — optional>
```

**type** — same set as §2.1, plus 3 commit-only types:

| type | Meaning |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Docs only |
| `refactor` | Refactor |
| `perf` | Performance |
| `test` | Tests only |
| `build` | Build system / external deps |
| `ci` | CI config |
| `chore` | Other misc |
| `revert` | Revert a prior commit |

**scope** — a top-level subpackage of `raven/`. See the `Repo layout` section of `README.md` for the canonical list. Spanning multiple scopes → omit the scope, or use `(*)`.

**subject** — lowercase start; no trailing period; English. The whole header (`<type>(<scope>): <subject>`) must be ≤ 100 chars — the single length rule, enforced by commitlint `header-max-length`.

**footer** (optional):
- `BREAKING CHANGE: <desc>` — triggers a MAJOR bump once public;
- `Closes #123` — auto-closes the issue on merge.

### §3.1.1 Top rule: the whole message is English (subject + body + footer)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EverMind-AI/Raven](https://github.com/EverMind-AI/Raven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
