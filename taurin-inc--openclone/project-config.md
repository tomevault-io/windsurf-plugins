---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository. Human-oriented docs are separate: [docs/architecture.md](docs/architecture.md) for the architecture walkthrough and [CONTRIBUTING.md](CONTRIBUTING.md) for PR process (both Korean).
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository. Human-oriented docs are separate: [docs/architecture.md](docs/architecture.md) for the architecture walkthrough and [CONTRIBUTING.md](CONTRIBUTING.md) for PR process (both Korean).

## What this repo is

A Claude Code **standalone skill** named `openclone`. The repo root **is** the skill — `SKILL.md` at the root declares the `/openclone` slash command and owns its dispatch logic. There is no build step, no test runner, no package manager, no `node_modules`. Distribution is by direct `git clone` into `~/.claude/skills/openclone/` (not `/plugin install`, not a marketplace), then `./setup` registers hooks + statusline in `~/.claude/settings.json`. Claude Code auto-discovers the skill on next session start.

## Commands

No `npm install`, no build — everything is bash and markdown. CI validators are TypeScript but Node 24+ runs `.ts` natively without flags.

```bash
./setup                                   # register UserPromptSubmit + SessionStart hooks + statusline in ~/.claude/settings.json (idempotent)
./uninstall                               # strip every _openclone_managed entry, delete ~/.claude/skills/openclone (keeps ~/.openclone)
./scripts/dev-link.sh <rel-path> [...]    # symlink workspace file(s) into installed skill — edits flow live
./scripts/dev-unlink.sh <rel-path> [...]  # remove dev-link; if the path is tracked, restore shipped version from git
touch ~/.openclone/no-auto-update         # disable SessionStart git pull (use while dev-linking)
rm ~/.openclone/no-auto-update            # re-enable auto-update
node .github/scripts/validate-skill.ts    # CI: SKILL.md frontmatter + references/*.md existence
node .github/scripts/validate-clones.ts   # CI: clones/*/persona.md schema + FIXED_CATEGORIES cross-file mentions
bash .github/scripts/smoke-hook.sh        # CI: hook JSON output across 5 states (no state, active, missing, room, force-push)
shellcheck hooks/*.sh scripts/*.sh        # CI shellcheck (severity: error; action also picks up setup/uninstall via shebang)
npx markdownlint-cli2 "**/*.md"           # CI markdownlint (config: .markdownlint-cli2.jsonc)
```

Node 22.6–23.5 requires `NODE_OPTIONS=--experimental-strip-types` to run the `.ts` validators.

## Project structure

```text
SKILL.md                       # single-dispatcher for /openclone — frontmatter + $ARGUMENTS branch table
README.md                      # user-facing install one-liner + usage (Korean)
CLAUDE.md                      # this file — AI-agent guide
CONTRIBUTING.md                # human contributor guide (Korean) — PR process, local dev loop, schema how-to
CHANGELOG.md · LICENSE · SECURITY.md · CODE_OF_CONDUCT.md
setup                          # bash; registers hooks + statusline in ~/.claude/settings.json + self-heals old installs
uninstall                      # bash; strips managed entries + removes install dir + cleans legacy plugin keys
clones/<name>/
  persona.md                   # built-in persona — shipped; sparse-default ON
  knowledge/                   # built-in knowledge — sparse-EXCLUDED; lazy-fetched on first /openclone <name>
hooks/
  inject-active-clone.sh       # UserPromptSubmit hook: room > active-clone > no-op; also emits force-push banner
scripts/
  session-update.sh            # SessionStart hook: fork-to-bg, throttled git pull --ff-only + cone→non-cone migration
  fetch-clone-knowledge.sh     # git sparse-checkout add clones/<slug>/knowledge — called by SKILL.md on activation
  statusline.sh                # renders "[display_name - role] 클론으로 대화중" or "[a, b, c +N] 클론들과 대화중"
  fetch-url.sh                 # curl + pandoc/html2text fallback when WebFetch is unavailable (ingest)
  fetch-youtube.sh             # yt-dlp transcript extractor (ingest; requires yt-dlp on PATH)
  dev-link.sh / dev-unlink.sh  # workspace → installed-skill symlink overlay for iteration
references/
  clone-schema.md              # SOURCE OF TRUTH for persona.md frontmatter/sections + knowledge filename rules
  categories.md                # the fixed 7 categories — vc, tech, founder, expert, influencer, politician, celebrity
  home-workflow.md             # /openclone (no arg) — home panel render + menu-context write
  interview-workflow.md        # /openclone new <slug>
  refine-workflow.md           # /openclone ingest <source>
  panel-workflow.md            # /openclone panel <category> "<question>" — also canonical "no emojis" rule
  room-workflow.md             # /openclone room — roster management + runtime routing rules
assets/clone-template.md       # copy-pasteable starting persona.md for hand-authoring
docs/architecture.md           # human-oriented Korean architecture walkthrough
.github/
  scripts/validate-skill.ts    # CI: SKILL.md frontmatter + body references/*.md existence check
  scripts/validate-clones.ts   # CI: persona.md schema + FIXED_CATEGORIES cross-file mentions (6 files)
  scripts/smoke-hook.sh        # CI: isolated-$HOME fixture — runs the hook across 5 states, asserts valid JSON + expected tags

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taurin-inc/openclone](https://github.com/taurin-inc/openclone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
