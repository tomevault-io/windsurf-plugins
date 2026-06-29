---
trigger: always_on
description: > **Status:** v4.0 (May 2026). Companion to `docs/roadmap.md` § "v4.0 — Agent-native author".
---

# Agents in SoloMD

> **Status:** v4.0 (May 2026). Companion to `docs/roadmap.md` § "v4.0 — Agent-native author".

SoloMD treats agents as **first-class editor surfaces**, not external CLI handoffs. This doc is the
*write-your-own-agent* guide: the YAML schema, what tools exist, the safety model, and the patterns
that work. If you just want to install a starter, open **Settings → Recipes → Browse cookbook**.

---

## TL;DR — three things to know

1. **A recipe is one YAML file** under `<workspace>/.solomd/agents/*.yml`. No build step, no plugin
   manifest. Edit, save, run.
2. **Every run is sandboxed** on its own AutoGit branch (`agent/<recipe>/<run-id>`). Writes don't
   touch `main` until you click **Accept** in Settings → Recipes → Pending.
3. **Read tools are free; write tools are gated** behind `allow-write: true` + a per-run `write-cap`
   (default 5, hard cap 50). The trace view shows every read and every write before you accept.

---

## YAML schema

```yaml
name: Weekly review              # required, displayed in UI
trigger: schedule                # required: schedule | on-save | on-commit | on-tag-add | manual
schedule: "0 18 * * SUN"         # required when trigger=schedule (5-field cron, UTC)
match: "daily/**/*.md"           # required for on-save / on-commit / on-tag-add (glob, workspace-relative)
tag: review-me                   # required for on-tag-add (no leading #)
prompt: |                        # required: the agent's instructions (multi-line block)
  Read this week's daily/ notes.
  Write weekly/{{date:YYYY-WW}}.md: themes / decisions / open threads.
allow-write: true                # default false — required if prompt asks for write_note / append_to_note
write-cap: 5                     # default 5, hard upper bound 50 — refusal bails before any side-effect
provider: claude                 # optional: claude | openai | anthropic | gemini | ollama | local | …
model: claude-sonnet-4-6         # optional, falls back to user's default model for the provider
base_url: https://api.x          # optional, only when self-hosting an OpenAI-compatible endpoint
tools:                           # optional explicit allowlist; default = every tool
  - read_note
  - write_note
```

A few subtleties worth knowing:

- **Cron is UTC**. `0 18 * * SUN` fires at 18:00 UTC on Sunday — that's 02:00 Monday in Beijing,
  10:00 Saturday in San Francisco. The cron parser accepts the standard 5 fields plus an optional
  6th (seconds, for testing).
- **Glob is workspace-relative** and uses the same `globset` crate as `gitignore` — `**/*.md` walks
  every folder, `daily/**/*.md` only inside `daily/`.
- **`tools:` allowlist** is a safety belt: when set, any tool the prompt asks for that's not in the
  list is refused at dispatch time. Useful for "this recipe should never write" recipes that you
  also mark `allow-write: false` for double protection.
- **`provider: local`** is an alias of `ollama` (kept for forward-compat with future local
  runtimes — see roadmap "Bundled local LLM runtime" → explicitly skipped).

---

## Triggers

| Trigger      | Fires when                                 | Required fields           | Notes |
|--------------|---------------------------------------------|---------------------------|-------|
| `schedule`   | Cron expression matches current UTC minute | `schedule:` (cron)        | Cron loop polls every 60s; never double-fires within a minute. |
| `on-save`    | A markdown file matching `match:` is saved | `match:` (glob)           | Debounced 800ms; rapid saves coalesce into one run. |
| `on-commit`  | An AutoGit commit lands and touches `match:` | `match:` (glob)           | Fires once per commit; multiple matched files = one run with `{{files}}` populated. |
| `on-tag-add` | A note gets `#tag` added that matches `tag:` | `tag:`                    | Detected by diffing the post-save tag list vs the pre-save list. |
| `manual`     | User clicks **Run now** in Settings → Recipes | (none)                    | The escape hatch — also useful for "test my prompt" cycles. |

---

## Prompt variables

The runner does Mustache-style substitution before sending the prompt to the model. Unknown
`{{tokens}}` are passed through verbatim, so embedding the literal string `{{example}}` in a prompt
is fine.

| Token             | Resolved to                                                            | Available in        |
|-------------------|-------------------------------------------------------------------------|---------------------|
| `{{path}}`        | Workspace-relative path of the triggering file                          | `on-save`, `on-tag-add` |
| `{{files}}`       | Comma-separated list of paths in the triggering commit                 | `on-commit`         |
| `{{sha}}`         | Short commit SHA                                                        | `on-commit`         |
| `{{tag}}`         | The `#tag` that was just added (no leading `#`)                         | `on-tag-add`        |
| `{{date:FORMAT}}` | Current date in any [chrono](https://docs.rs/chrono) format string. Common: `YYYY-MM-DD`, `YYYY-WW` (ISO week), `YYYY-MM` | All triggers |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhitongblog/solomd](https://github.com/zhitongblog/solomd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
