---
trigger: always_on
description: This is the orientation document for AI agents (and humans) working in this repo. It exists so an agent dropped in cold can write idiomatic dcs-sms code without grepping every module first.
---

# AGENTS.md — dcs-sms framework reference for AI agents

This is the orientation document for AI agents (and humans) working in this repo. It exists so an agent dropped in cold can write idiomatic dcs-sms code without grepping every module first.

It is deliberately a **rules-and-conventions** document, not an API reference. Every public `sms.*` symbol is documented per-module with worked examples in [`docs/api/`](docs/api/) — that is the canonical reference. Read the relevant `docs/api/<module>.md` page before you write code that touches that module.

> **Companion documents:**
> - [`docs/api/`](docs/api/) — per-module reference: signatures, options tables, runnable examples, see-also.
> - [`docs/superpowers/specs/`](docs/superpowers/specs/) — per-module design docs (canonical "why is it shaped this way").
> - This file is a *summary*. When the spec disagrees with this file, the spec wins.

---

## 1. The prime directive

**Always prefer `sms.*` over vanilla DCS API when the framework provides it.**

The framework exists precisely because vanilla DCS scripting is awkward, undocumented, and full of footguns. If a vanilla call would replace a single line of `sms.*` code, use the `sms.*` version every time.

**When the framework does not yet cover what you need:**

1. Use the vanilla DCS API (`Group.getByName`, `coalition.addGroup`, `world.addEventHandler`, `trigger.action.*`, etc.) to get the job done.
2. **Surface this to the user.** Say something like:
   > "I needed to call vanilla DCS `<api>` here because `sms.<module>` doesn't expose this yet. Want me to file a GitHub issue so we can fold this into the framework later?"
3. If the user agrees, invoke the `make-issue` skill (or use `gh issue create`) with a description of the gap, the use case, the vanilla API used, and a sketch of what the `sms.*` shape might look like.

This is non-negotiable. The framework only grows by noticing where it falls short. **If you silently fall back to vanilla without flagging the gap, you have failed the assignment.**

### What counts as "the framework doesn't cover it"

- The function does not exist on any `sms.*` module.
- The function exists but its current contract doesn't expose the data you need (e.g. `sms.unit.get_position` returns vec3 but you need the full Position3 with orientation axes).
- The function exists but a known issue (`docs/superpowers/specs/`, GitHub issues) limits it.

### What does NOT count

- "There's a vanilla one-liner that's slightly shorter." Use `sms.*` anyway — the framework's value is the *consistency* and the failure model, not the syntactic sugar.
- "The framework version returns nil on bad input but I want it to throw." This is the framework's [failure model](#3-failure-model-log--nil-never-throw) by design. Never bypass it.

---

## 2. Repo layout at a glance

- **`framework/`** — in-DCS Lua framework. Runs inside the mission environment. One file per `sms.*` module, plus `load_all.lua` (one-shot loader) and `test/` (bash smoke tests driven by `tools/dcs-sms.exe`).
- **`tools/`** — host-side Go. Produces the `dcs-sms` CLI and the embedded `Scripts/Hooks` Lua hook. Not loaded into DCS missions; talks to them via a filesystem mailbox.
- **`docs/api/`** — per-module reference pages with worked examples. Canonical source for "how do I call this".
- **`docs/superpowers/specs/`** — design docs, one per sub-project / module. Canonical source for "why is it shaped this way".

**Two distinct Lua environments:**

- **Mission environment** (`framework/*.lua`): sandboxed by `Scripts/MissionScripting.lua`. `os`, `io`, `lfs` are nilled. Lua 5.1. `print` is silent — use `env.info` / `env.error` (or `sms.log.*`).
- **Hook environment** (`tools/lua/dcs-sms-hook.lua`): NOT sandboxed. Has LuaSocket, `lfs`, full file I/O. Used by the host-side bridge.

The framework runs in the mission environment. The bridge runs in the hook environment. They communicate through the filesystem mailbox the bridge installs.

---

## 3. Failure model: log + nil, never throw

**Every public framework call follows this contract:**

- On bad input or missing entity → log via the module's tagged logger and return `nil` or `false`.
- **Never `error()` out of an `sms.*` call.** Throwing aborts the entire mission script — that is the failure mode dcs-sms exists to avoid.
- Methods accept either a handle (`{name=...}` table) or a raw name string interchangeably. They also tolerate garbage (nil, numbers, booleans) — those normalize to "not alive" and produce a logged nil.

```lua
-- All four of these behave the same way: log + return nil if the unit isn't there.
sms.unit("ghost"):get_position()
sms.unit.get_position("ghost")
sms.unit.get_position({name = "ghost"})
sms.unit.get_position(nil)
```

### Log levels: warn for caller misuse, error for real failures

`sms.log` exposes four levels — `debug`, `info`, `warn`, `error` — each landing on the corresponding `env.*` sink (`env.info`, `env.info`, `env.warning`, `env.error`). Pick the right one when authoring framework code:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nielsvaes/dcs-sms](https://github.com/nielsvaes/dcs-sms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
