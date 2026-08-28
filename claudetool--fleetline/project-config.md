---
trigger: always_on
description: This file exists so a fresh Claude Code session (different machine, no
---

# fleetline — project context for Claude

This file exists so a fresh Claude Code session (different machine, no
memory of the original conversation) can pick this project back up with
full context. Read this before touching any code here.

## What this is

A Claude Code statusline plugin, distributed two ways: the plugin
marketplace (`.claude-plugin/plugin.json` + `commands/statusline-setup.md`
does the setup interview and writes config/settings.json) and a
standalone `install.sh` (curl-installable, no marketplace needed, but
missing the hook-based and fleet-view features — see below).

Originally started as the user's own personal statusline script
(`~/.claude/statusline-command.sh`), shared informally as a one-file
installer. This repo is that script turned into a proper, hardened,
shareable plugin — first named `claude-statusline`, renamed to
`fleetline` after discovering the original name was already taken by
an unrelated project (verified via web search: felipeelias.github.io
blog post, plus `ccstatusline`/`starship-claude`/`cship`/`ccsidekick`/
`codachi` already crowding the space).

## File map

```
.claude-plugin/plugin.json    plugin manifest (name, version, description)
bin/lib.sh                    shared helpers — colors, sanitize(), cfg_bool(),
                               draw_bar()/rate_bar(), fit_line() (width-aware
                               truncation), OSC-8 link builders. Sourced by
                               both scripts below; not executed directly.
bin/statusline.sh             the main statusLine renderer (1-N lines, no
                               fixed count). Every piece is computed into a
                               *_SEG/*_PART variable, looked up by id via
                               segment_value(). Lines come from either the
                               layout's built-in default (3 fixed presets) or
                               the config's `segments` override — a FLAT
                               ordered id array with a reserved "newline" id
                               to break lines, modeled after powerlevel10k's
                               POWERLEVEL9K_LEFT_PROMPT_ELEMENTS — see
                               "Custom segment order" in README.md. The
                               separator glyph between segments is also
                               config-driven (`separator.preset`/`.custom`).
bin/subagent-statusline.sh    the subagentStatusLine renderer (fleet/agent
                               panel view) — LEAST-VERIFIED file, see below
hooks/hooks.json + agent-count.sh   optional SubagentStart/Stop counter,
                               plugin-only (see "Open questions")
config/schema.json            documents every config field + default
commands/statusline-setup.md  the /statusline-setup interview + settings.json
                               writer (with backup-before-overwrite)
install.sh                    curl-installable single-file build — bin/lib.sh
                               is INLINED into it (regenerate if you edit
                               lib.sh or statusline.sh — see below)
docs/index.html               GitHub Pages landing page (self-contained,
                               dark-only by deliberate choice, no build step)
README.md                     user-facing docs (install, config, limitations)
```

## Non-obvious things you need to know before editing

**If you edit `bin/lib.sh` or `bin/statusline.sh`, you MUST regenerate
`install.sh`'s inlined copy** — it's not a symlink or an `include`, it's a
literal copy-paste inside a heredoc (`cat > "$SCRIPT_DEST" << 'STATUSLINE_EOF'`)
because the curl-install path writes one self-contained file and can't
rely on a sibling `lib.sh` existing. The regen recipe (works from repo root):

```bash
grep -n "STATUSLINE_EOF" install.sh   # confirm both marker line numbers before trusting the two sed calls below — they drift every time the inlined content changes size (currently 37 open, 582 close)
{ sed -n '1,37p' install.sh; } > /tmp/header.sh
{ sed -n '583,$p' install.sh; } > /tmp/footer.sh
{
  echo '#!/usr/bin/env bash'
  echo '# fleetline — pluggable Claude Code statusline (curl-install build: lib.sh'
  echo '# inlined below since this installer writes a single self-contained file).'
  echo
  tail -n +2 bin/lib.sh
  echo
  sed -n '2,17p;22,$p' bin/statusline.sh
} > /tmp/inlined.sh
cat /tmp/header.sh /tmp/inlined.sh > install.sh; echo STATUSLINE_EOF >> install.sh; cat /tmp/footer.sh >> install.sh
bash -n install.sh   # then diff-test 2-file vs inlined output on the same payload before trusting it
```
The `2,17p;22,$p` range in statusline.sh skips its own shebang and the
`source ./lib.sh` lines (currently lines 18-21) — re-check those line
numbers if you've edited the top of the file.

**jq's `//` treats a stored `false` the same as `null`/missing.** This bit
us twice: once in config reading (fixed via `cfg_bool()` in lib.sh — never
use `cfg '.foo // true'` for a boolean, always `cfg_bool '.foo' true`),
and once in stdin JSON reading (`.thinking.enabled // empty` silently
turned an explicit `false` into "absent", so "thinking:off" could never
render — fixed by switching to an explicit null-check in the jq filter).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClaudeTool/fleetline](https://github.com/ClaudeTool/fleetline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
