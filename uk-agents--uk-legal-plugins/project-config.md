---
trigger: always_on
description: Guidance for working on this repo. `uk-legal-plugins` is a UK-jurisdiction
---

# CLAUDE.md

Guidance for working on this repo. `uk-legal-plugins` is a UK-jurisdiction
legal plugin marketplace — 11 plugins, 140+ skills. Most work here is editing
prompt content (skills, agents, hooks) or plugin metadata — not application code.

## Target ecosystems

These plugins reach **three** MCP-speaking client ecosystems via dual-manifest packaging (`.claude-plugin/plugin.json` + `.codex-plugin/plugin.json` per plugin, sharing the same `skills/` + `.mcp.json`):

| Ecosystem | Audience | Reaches via |
|---|---|---|
| Claude Code | Developer + lawyer-developers | `.claude-plugin/plugin.json` |
| Cowork (Anthropic) | Anthropic collaborative env | Same as Claude Code |
| OpenAI Codex CLI | OpenAI developer audience | `.codex-plugin/plugin.json` |

ChatGPT consumer (~80M/week) is NOT reachable through these plugins — ChatGPT supports MCP tools only, no skills, no resources, no prompts. ChatGPT users connect directly to `uk-legal-mcp.fly.dev/mcp`; for them, the MCP server's tool descriptions are the only workflow-teaching layer. See `uk-legal-mcp/docs/chatgpt-workflow-encoding.md` (in the sibling repo).

## Documentation index

In `docs/`:

- [`SPEC.md`](docs/SPEC.md) — pre-existing specification
- [`skill-gaps-and-design.md`](docs/skill-gaps-and-design.md) — 5 new skills closing the dogfeed-failure workflows (find-member-contribution, find-case-by-party-verify, oscola-build-citation, statute-amendments-trace, bill-debate-trace)
- [`distribution-strategy.md`](docs/distribution-strategy.md) — distribution decision (Option C1: single free public repo), dual-manifest plugin pattern, future revenue paths (recorded; not implemented)

## Layout

```
.claude-plugin/marketplace.json   # the marketplace manifest — one entry per plugin
<plugin>/                         # 11 plugins (employment-legal-uk, privacy-legal-uk, ...)
  .claude-plugin/plugin.json      # plugin manifest (name, version, description, author)
  .mcp.json                       # MCP servers the plugin connects to
  CLAUDE.md                       # practice-profile TEMPLATE (see "Plugin CLAUDE.md" below)
  README.md                       # per-plugin docs
  skills/<name>/SKILL.md          # one skill per directory
  agents/<name>.md                # subagent definitions
  hooks/hooks.json                # hook config (most plugins ship an empty stub)
  .gitignore
references/                       # shared templates read by cold-start and dashboard skills
  company-profile-template.md     # shape of ~/.claude/plugins/config/uk-legal-plugins/company-profile.md
  dashboard-template.md           # rendering standard for HTML/terminal/Excel dashboard outputs
```

## Validation — run before opening a PR

This repo follows the same conventions `anthropics/claude-plugins-official`
enforces in CI. Run the equivalent checks locally:

```bash
# 1. Marketplace + per-plugin schema validation (source of truth)
claude plugin validate .claude-plugin/marketplace.json
for d in */; do [ -f "$d/.claude-plugin/plugin.json" ] && claude plugin validate "$d"; done

# 2. JSON sanity
python3 -c "import json,glob; [json.load(open(f)) for f in glob.glob('**/*.json', recursive=True)]"
```

### Marketplace invariants (I1–I11)

`claude-plugins-official` layers these on top of the schema check. They apply
here too — the ones most likely to trip a contributor:

- **I1** — `plugins[]` should be alpha-sorted by name (case-insensitive).
  *Currently a known warning: the array is in a curated display order. If you
  add a plugin, ask before re-sorting the whole array.*
- **I2** — no duplicate plugin names.
- **I3** — `description` 10–2000 chars, no leading/trailing whitespace.
- **I8** — every vendored `source` (`"./<dir>"`) must point at a directory that
  contains `.claude-plugin/plugin.json`.
- **I9** — `source` paths/URLs must contain no shell metacharacters or `..`.
- **I10** — no hidden Unicode (zero-width chars, bidi controls) in
  `name`/`description`.
- **I11** — `name` must match `^[a-z0-9][a-z0-9-]{1,63}$`.

### Frontmatter requirements

Every `agents/*.md` needs `name` and `description`. Every
`skills/<name>/SKILL.md` needs `description`. Every `commands/*.md` needs
`description`. Multi-line descriptions use `>` block scalars and that's fine —
`claude plugin validate` parses them correctly.

## Skill authoring conventions

### MCP-native skill pattern (reference: `regulatory-legal-uk:reg-feed-watcher`)

Skills that interact with `uk-legal-mcp` (or any other MCP server in `.mcp.json`) follow five characteristics, taken from `regulatory-legal-uk:reg-feed-watcher` which is the reference template:

1. **"Pushy" trigger description.** YAML frontmatter `description:` opens with "USE WHEN the user says X / asks Y" — concrete user phrases. Agents systematically under-trigger when the description is passive ("a skill for searching"). Use `skill-creator`'s description-optimization loop on every new skill.

2. **MCP tools named verbatim in prose.** Skill instructions write actual tool names: "Call `parliament_find_member(name)` to get the member_id." Not "look up the member." The agent matches actual tool names — named-verbatim wins.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uk-agents/uk-legal-plugins](https://github.com/uk-agents/uk-legal-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
