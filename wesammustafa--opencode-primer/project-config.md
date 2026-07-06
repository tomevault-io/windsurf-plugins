---
trigger: always_on
description: > Project instructions for OpenCode. Read every session.
---

# AGENTS.md

> Project instructions for OpenCode. Read every session.

## What this repo is

A practical, opinionated guide to using [OpenCode](https://opencode.ai) — README, supporting docs, drop-in agent prompts, MCP walkthroughs, and a worked example of a fully-configured `.opencode/` directory. It's a **documentation repository**, not application code.

## Repo layout

```
.
├── README.md                       # Main guide (the centerpiece)
├── AGENTS.md                       # This file
├── CONTRIBUTING.md                 # How to contribute
├── SECURITY.md · CODE_OF_CONDUCT.md
├── LICENSE                         # MIT
├── opencode.json · tui.json        # Example configs with safe defaults
├── docs/
│   ├── quickstart.md               # 5-minute getting-started
│   ├── zen.md                      # Zen model gateway details
│   ├── tui.md                      # TUI mastery
│   ├── agents.md                   # Custom agent deep dive
│   ├── commands.md                 # Custom slash commands
│   ├── skills.md                   # Agent skills
│   ├── plugins.md                  # Plugin/hook system
│   ├── mcp.md                      # MCP integration
│   ├── workflows.md                # Real-world recipes
│   ├── migration.md                # Migration from Claude Code / Cursor
│   └── reference/
│       ├── cli.md                  # All CLI commands
│       ├── slash-commands.md       # All TUI slash commands
│       ├── permissions.md          # Permission model
│       ├── changelog.md            # OpenCode releases
│       ├── faq.md                  # FAQ + troubleshooting
│       └── further-reading.md      # External resources
├── .opencode/
│   ├── agents/                     # Example custom agents
│   ├── commands/                   # Example custom commands
│   ├── plugins/                    # Example plugins (audit log, secret blocker)
│   └── skills/                     # Example agent skills
├── .github/                        # CI (lint-docs.yml), issue/PR templates
├── mcp-servers/                    # MCP server walkthroughs
├── specialized-agents/             # Drop-in specialist prompts
│   ├── system-prompts/             # System prompts (7 roles)
│   └── descriptions/               # Role descriptions
└── Images/                         # Banner, diagrams, screenshots
```

## Editing rules

### Always

- **Cite the source.** Every factual claim about OpenCode must trace back to [opencode.ai/docs](https://opencode.ai/docs/), the [GitHub repo](https://github.com/anomalyco/opencode), or an official release note. Don't paraphrase from memory.
- **Verify version-specific claims.** Pricing, model names, commands, and flags change. When in doubt, check the docs page that owns the topic.
- **Use OpenCode's actual terminology.** "Custom commands" not "slash skills"; "agents" and "subagents" not "personas"; "plugins" not "hooks" (OpenCode plugins serve the hook role).
- **Match the README's tone.** Practical, mental-model-first, prescriptive ("Reach for it when…"). No marketing fluff.
- **Update the `Last reviewed` date** at the bottom of the README when making material changes.

### Never

- **Don't invent commands, flags, or features.** If you can't verify it, leave it out.
- **Don't hardcode prices without a "verified" date.** Always show the date next to any pricing table.
- **Don't add `Co-Authored-By` lines to commits** unless the user asks for them.
- **Don't `git push --force` to `main`**, ever, without explicit permission.
- **Don't introduce affiliate links or sponsorship blocks** without the maintainer's approval.

## Style conventions

### Markdown

- One blank line between sections. No trailing whitespace.
- Code fences with language tags: ` ```bash `, ` ```json `, ` ```markdown `.
- Internal links use anchor form: `[label](#section-id)`. Anchors are auto-derived from headings (lowercased, spaces → dashes, special chars stripped).
- Tables: header row, separator row, no leading/trailing pipes. Keep column widths reasonable for desktop reading.
- Callouts use blockquotes with emoji prefixes:
  - `> 💡 **Pro Tip:** …` for tips
  - `> ⚠️ **Warning:** …` for foot-guns
  - `> 🆕 …` for new features
  - `> 📚 …` for deeper-reading pointers
- Emoji policy:
  - **OK** — one leading emoji on H2 navigation headings (🧭 🧠 📚 📖), and the callout-prefix set above.
  - **Not OK** — decorative emoji inside prose, in code, or on H3/H4 headings. One per spot, never decorative density.

### Code samples

- **JSON examples** in this repo are valid `opencode.json` snippets — paste-ready. Include the `$schema` line on full-file examples.
- **Shell examples** assume a POSIX shell. Use `bash` fence; quote variables (`"$VAR"`).
- **Markdown front-matter** uses YAML with `---` fences. Match the exact key spelling from the OpenCode docs (e.g., `description`, `mode`, `model`, `permission`, `tools`).

### Tone

- **Be concrete.** "Reach for it when X" beats "useful for X."
- **Lead with the mental model**, then the syntax, then the example.
- **Compare options side-by-side** when there's a choice to make (custom commands vs. skills, etc.).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wesammustafa/opencode-primer](https://github.com/wesammustafa/opencode-primer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
