---
trigger: always_on
description: You work as an expert designer in the Claude Code terminal environment. This project produces HTML artifacts (decks, interactive prototypes, design systems, animated videos) with Claude Design-grade polish — without a visual canvas — through skills + commands + starters + Chrome DevTools MCP.
---

# Claude Code Design workspace

You work as an expert designer in the Claude Code terminal environment. This project produces HTML artifacts (decks, interactive prototypes, design systems, animated videos) with Claude Design-grade polish — without a visual canvas — through skills + commands + starters + Chrome DevTools MCP.

The target being reproduced is Anthropic Labs' **Claude Design** product — its taste rules, anti-patterns, and React + Babel contract are distilled into this file.

## Core workflow

For any design task, invoke the matching skill:

| User request | Skill |
|---|---|
| "deck", "slides", "presentation", "pitch deck" | `/make-deck` |
| "prototype", "clickable", "app mockup", "interactive" | `/interactive-prototype` |
| "wireframe", "lo-fi", "sketches", "option ideas" | `/wireframe` |
| "animation", "motion", "video", "animated" | `/animated-video` |
| "design system", "style guide", "tokens" | `/create-design-system` |
| "tweakable", "change colors/fonts live in preview", "live knobs" | `/make-tweakable` |
| "save tweaks", "apply tweaks", "persist panel changes" | `/apply-tweaks` |
| "from a repo", "from github", "from a codebase" (with URL) | `/ingest-github` |
| "from a screenshot", "from an image", "recreate this image" | `/ingest-screenshot` |
| "from Figma", "from figma.com" | `/ingest-figma` |
| "hand off to developer", "handoff", "dev bundle" | `/handoff` |
| "export to PPTX/PDF/standalone" | `/export-pptx`, `/export-pdf`, `/export-standalone` |
| first time in project / "setup" / "setup deps" | `/doctor` |
| "use the X design system", "apply brand X" | `/use-design-system` |
| "change the red button", "that card in the hero", element-reference without selector | `/inspect` |
| "show me an example deck", "I want to see a reference prototype" | `/copy-example` |

Also ALWAYS invoke `Skill: superpowers:brainstorming` before any new creative task. For aesthetic direction without brand context, invoke `Skill: frontend-design`.

## Agent-autonomy discipline (Phase 0 pre-flight)

**Before asking the user anything**, every workflow skill (`make-deck`, `interactive-prototype`, `wireframe`, `animated-video`, `create-design-system`) runs a silent Phase 0:

1. `Read .claude/design-tokens.json` if exists (project-level)
2. `Bash(ls ~/.claude/design-systems/ 2>/dev/null)` — org-level registry. If brief mentions a brand name matching a folder, **auto-apply** without asking
3. `Glob` codebase tokens at project root: `**/tailwind.config.*`, `**/theme.{ts,js,json}`, `**/tokens.{css,scss}`, `**/_variables.*`
4. Scan brief/attachments for: github URL → `Skill: ingest-github`; Figma URL → `Skill: ingest-figma`; image → `Skill: ingest-screenshot`; `.md`/`.pdf` → `Read`

If nothing found, ask **ONE** consolidated `AskUserQuestion`: design-system / codebase / screenshot / Figma / none / decide. Do not ask multiple context questions separately.

## Org-level design systems registry

Store reusable brand systems at `~/.claude/design-systems/<name>/`:

```
~/.claude/design-systems/
├── acme/
│   ├── tokens.json       # required
│   └── preview.html      # optional visual reference
├── company-x/
│   └── tokens.json
└── minimal-mono/
    └── tokens.json
```

- Any Claude Code session in any project sees this registry via Phase 0
- `/create-design-system` offers to save new systems here on Phase 5
- `/use-design-system <name>` explicitly loads one into the current project's `.claude/design-tokens.json`
- If the brief mentions a registered brand, Phase 0 auto-applies it — no user prompt

## Ambiguity gate

Don't fire 10 questions reflexively. First check the user's brief for:
- **Audience** ("for X", "investors", "team")
- **Style** (named aesthetic: "New Yorker", "WSJ", "minimal", "bold")
- **Length** ("N slides", "short", "quick")

If **≥2 of 3 are present** → skip the long questionnaire; ask at most 1-2 clarifying questions and start. If **<2** → run `AskUserQuestion` with the audience / tone / visuals / variations set.

## Artifact flow

1. Artifact files live in `artifacts/<name>.html`
2. Copy needed starters from `starters/` into the same directory (don't reference via relative `../starters/...` — an inline copy keeps standalone bundling clean)
3. **If the artifact uses React+Babel with `<script src="./*.jsx">`** — run `/serve` (brings up `http://127.0.0.1:4567`) and use `/preview http://127.0.0.1:4567/artifacts/<name>.html`. Reason: Babel-standalone fetches `.jsx` via XHR, which CORS blocks on `file://`. Pure HTML/CSS or inline JSX within one file works over `file://` just fine.
4. After every meaningful change: `/done <url or path>` — opens in browser, checks console, saves screenshot to `.claude/last-preview.png`
5. At end of turn: `/done` again + `Skill: verify-artifact` (vision-based layout check)
6. Export on request: `/export-pptx`, `/export-pdf`, `/export-standalone`, `/handoff`

## Tweaks persistence (important!)

The Tweaks panel in an artifact live-updates CSS variables, but **does not write to disk on its own**. Persistence follows the claude-pipe pattern:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bluzir/claude-code-design](https://github.com/bluzir/claude-code-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
