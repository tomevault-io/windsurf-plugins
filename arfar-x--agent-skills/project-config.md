---
trigger: always_on
description: This repo holds AI-agent skills (Hermes, Claude Code, claude.ai), not an
---

# Agent instructions

This repo holds AI-agent skills (Hermes, Claude Code, claude.ai), not an
application. There is no build step and no server to run -- "testing a
change" means running the relevant toolset's pytest suite. The repo is
meant to hold multiple unrelated toolsets over time (Jira today, e.g. a
company back-office toolset later), each following the same layout.

## Repo layout

Not every skill belongs to a toolset. A **standalone skill** (e.g.
`skills/mood/`) is a single `SKILL.md` file with no sibling toolset
directory, no `lib/`/`tools/`/`scripts/`/`tests/`, and nothing to
`pip install` -- pure instructions the agent follows directly. The
toolset shape below is the common case in this repo, not a requirement
every skill must satisfy. `skills/mood/` in particular defaults to
`neutral` -- the agent's normal tone -- unless the user has explicitly
switched to another mode in the current conversation.

A toolset (or standalone skill) can also be marked **internal** --
`metadata.internal: true` in its `SKILL.md` frontmatter -- to keep it out
of normal installs and listings; it only appears when the installer is
run with `INSTALL_INTERNAL_SKILLS=1`. This is for a skill whose risk
profile doesn't belong in a default install (e.g. `skills/telegram/`,
which grants standing access to a personal account) -- it is not a
general-purpose "hide this skill" switch, and it doesn't relax any of the
conventions below.

A skill can separately set `metadata.mcp: false` in its `SKILL.md`
frontmatter to keep it out of `mcp-server`'s exposure specifically,
without affecting anything else -- a non-internal skill with `mcp: false`
still installs and lists normally everywhere else (`npx skills`, Hermes,
Claude Code, claude.ai); it just never becomes an MCP tool or shows up in
`mcp-server`'s `list_skills`/`get_skill`. Default is `true` (exposed) when
the key is absent -- only set it to `false` when a skill genuinely
shouldn't be reachable over MCP (e.g. one whose whole point is a runtime
capability MCP has no equivalent for). Internal and `mcp: false` land on
`mcp-server` the same way -- neither is exposed there -- but they differ
everywhere else: an internal skill is also hidden from `npx skills`
`--list`/`--all` (installable only by naming it explicitly with
`INSTALL_INTERNAL_SKILLS=1`), while `mcp: false` has no effect on `npx
skills` at all. And unlike `internal`, there's no override flag for `mcp:
false` -- `--include-internal` brings an internal skill's tools back, but
nothing brings back a skill whose own frontmatter says it isn't meant for
MCP.

A standalone skill that produces a document from a template (`prd`,
`trd`, `adr`, `rfc`, `agents-md`, and any future `erd`/...) should
additionally set `metadata.doc_type: <slug>` in its `SKILL.md`
frontmatter -- see `skills/prd/SKILL.md`, `skills/trd/SKILL.md`,
`skills/adr/SKILL.md`, `skills/rfc/SKILL.md`, and
`skills/agents-md/SKILL.md`. This is what
[`mcp-server/`](mcp-server)'s `doc_gen` tool discovers at startup to
build its `doc_type` enum; a new document-generation skill only needs
this one frontmatter line to appear there automatically, with no
`mcp-server` code changes. Don't set it on a standalone skill that isn't
a document template (`mood` doesn't set it).

A toolset also isn't required to ship thin per-action `skills/<toolset>-*/`
wrapper skills (below) -- `skills/telegram/` is the first example of a
toolset with none, deliberately, since multiplying a high-risk skill's
installable surface across ten separate entry points is itself a risk to
avoid, and an internal skill isn't being exposed as a discoverable command
catalog anyway. The wrapper pattern is the norm for a toolset meant to be
installed piecemeal, not a requirement every toolset must satisfy.

Per toolset `<toolset>` (e.g. `jira`):

- `skills/<toolset>/` -- the actual implementation: `lib/` (REST client,
  env-based config), `tools/` (thin per-action entry points),
  `scripts/<toolset>_tool.py` (CLI dispatcher), `tests/`.
- `skills/<toolset>-*/` -- thin `SKILL.md`-only skills, one per action,
  that shell out to `../<toolset>/scripts/<toolset>_tool.py`. They exist
  purely so each action gets its own Hermes slash command; they contain
  no Python of their own and nothing to test.

**`skills/_shared/`** holds code more than one toolset needs -- e.g.
`credentials/http.py`'s `Credential`/`BasicCredential`/`BearerCredential`
types, used identically by any toolset authenticating a `requests.Session`
(Jira today). A toolset that needs one of these files **symlinks** it
into its own `lib/` (`ln -s ../../_shared/credentials/http.py
skills/jira/lib/credentials.py`) rather than copying it -- one edit
updates every consumer. This works under both consumption paths this
repo supports: a direct `git clone` preserves the symlink natively, and
`npx skills add --skill <name>`'s own installer (`vercel-labs/skills`)
copies a symlinked file with `dereference: true` specifically to handle
exactly this case (confirmed by reading its actual source, not assumed).
**`skills/_shared/` must never contain a `SKILL.md`** -- both consumers
above discover skills by walking for `SKILL.md`'s presence, so a
directory without one is structurally invisible to either, not just

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arfar-x/agent-skills](https://github.com/arfar-x/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
