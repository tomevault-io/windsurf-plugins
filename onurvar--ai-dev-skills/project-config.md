---
trigger: always_on
description: A Claude Code plugin marketplace for iOS, Android, and React Native development. Users add it via `/plugin marketplace add <this-repo>` and install the stack-specific plugins they need.
---

# ai-dev-skills

A Claude Code plugin marketplace for iOS, Android, and React Native development. Users add it via `/plugin marketplace add <this-repo>` and install the stack-specific plugins they need.

This file is the **reference point for contributors** working on this repo. It captures the design decisions we've locked in — purpose, structure, principles, review pipeline, how to add stacks, what to include and what to exclude. Read this before adding, removing, or changing any plugin.

## What this repo is

Six plugins — five per-stack, one platform-neutral:

| Plugin | Covers | Skills | Status |
|---|---|---|---|
| `ov-ios` | Native iOS (Swift / SwiftUI / UIKit / Obj-C) | init, plan, review-deep | Rules file drafted; skills scaffolded |
| `ov-android` | Native Android (Kotlin / Java / Jetpack Compose) | init, plan, review-deep | Rules file drafted; skills scaffolded |
| `ov-rn-typescript` | React Native TypeScript / JS layer | init, plan, review-deep | Rules file drafted; skills scaffolded |
| `ov-rn-ios-native` | React Native iOS bridge (Swift / Obj-C) | init, plan, review-deep | Rules file drafted; skills scaffolded |
| `ov-rn-android-native` | React Native Android bridge (Kotlin / Java) | init, plan, review-deep | Rules file drafted; skills scaffolded |
| `ov-pr` | Any stack — lightweight GitHub PR review | review-quick | Drafted |

**Naming convention:** every plugin is `ov-` prefixed so that typing `/ov` in Claude Code lists exactly these commands and nothing else. The two review commands are named for their depth, not gated behind a flag: `review-deep` is the multi-tool pipeline over your own pending changes, `review-quick` is one pass over someone's open PR.

## Installation

Two ways, depending on your environment:

**Option A — Claude Code plugin marketplace (`/plugin`, when permitted):**

```
/plugin marketplace add /path/to/ai-dev-skills
/plugin install ov-ios@ai-dev-skills
# repeat for ov-android, ov-rn-typescript, ov-rn-ios-native, ov-rn-android-native, ov-pr
```

Skills are invoked as `/ov-ios:init`, `/ov-ios:plan`, `/ov-ios:review-deep`, `/ov-pr:review-quick`, etc. (plugin-namespaced with a colon).

**Option B — filesystem install via `./install.sh`** (when `/plugin` is forbidden or unavailable):

```
./install.sh              # symlink (default) — edits to this repo propagate
./install.sh --copy       # snapshot copy — self-contained
./install.sh --uninstall  # remove everything the script installed
./install.sh --help       # full usage
```

Skills are invoked as `/ov-ios-init`, `/ov-ios-plan`, `/ov-ios-review-deep`, `/ov-pr-review-quick`, etc. — colon becomes dash since user-level skills aren't plugin-namespaced. Rules files land at `~/.claude/ai-dev-skills/rules/<stack>.md` (filenames stay unprefixed; the prefix exists for command discovery, and rules files are never invoked). Each stack's `init` skill resolves both the plugin path and this manual path so either install mode works.

Re-running `./install.sh` after a rename prunes the entries an earlier install created that the current one no longer produces, so stale symlinks don't accumulate in `~/.claude/skills/`.

## Per-plugin structure

Per-stack plugins are identical to each other:

```
plugins/ov-<stack>/
├── .claude-plugin/plugin.json
├── rules/<stack>.md                  ← always-on rules, imported into user's CLAUDE.md
└── skills/
    ├── init/SKILL.md                 ← /ov-<stack>-init        — bootstraps @import into project CLAUDE.md
    ├── plan/SKILL.md                 ← /ov-<stack>-plan        — planning ritual
    └── review-deep/SKILL.md          ← /ov-<stack>-review-deep — review pipeline
```

The platform-neutral plugin has no rules file — it carries no standing stack knowledge:

```
plugins/ov-pr/
├── .claude-plugin/plugin.json
└── skills/review-quick/SKILL.md      ← /ov-pr-review-quick     — one-pass PR review
```

**Why this shape:**

- **`rules/<stack>.md`** — always-on knowledge. When imported into a project's CLAUDE.md, every Claude session in that project loads it automatically. It is also load-bearing for review: `/pr-review-toolkit:review-pr`, `/security-review`, `/simplify`, `/code-review:code-review` all read CLAUDE.md to know what rules to apply.
- **`skills/init/`** — the bootstrap ritual. A plugin cannot modify the user's project CLAUDE.md directly, so `/ov-<stack>-init` copies the rules file into `.claude/<stack>-rules.md` and appends an `@import` line to the project's CLAUDE.md.
- **`skills/plan/`** — the before-implementation ritual.
- **`skills/review-deep/`** — the after-implementation ritual; orchestrates the full pipeline.
- **`skills/review-quick/`** — the read-someone-else's-PR ritual; deliberately not a pipeline.

Skill frontmatter carries `description` only, no `name` — the directory name is what becomes the command, in both install modes.

## Design principles (non-negotiable)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OnurVar/ai-dev-skills](https://github.com/OnurVar/ai-dev-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
