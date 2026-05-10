---
trigger: always_on
description: Claude Message is a dynamic messaging system for Claude Code. It provides the harness that helps product marketers build, maintain, and operationalize their positioning and messaging across the entire GTM lifecycle. The messaging house in `messaging/` is the single source of truth — agents read it for context, write to it with user approval, and generate content from it. Users run `/bootstrap` to build the messaging system from scratch.
---

# CLAUDE.md

## About Claude Message

Claude Message is a dynamic messaging system for Claude Code. It provides the harness that helps product marketers build, maintain, and operationalize their positioning and messaging across the entire GTM lifecycle. The messaging house in `messaging/` is the single source of truth — agents read it for context, write to it with user approval, and generate content from it. Users run `/bootstrap` to build the messaging system from scratch.

## Writing Profile

Generate consistent, clear, and compelling messaging grounded in the company's market position, audience, and differentiation.

<!-- claude-message:profile:start -->
Run `/bootstrap` to generate your writing profile from the messaging house.
<!-- claude-message:profile:end -->

When a writing profile exists between the markers above, use it as the primary context for all messaging work. The profile specifies the company identity, stage, and market — use these to calibrate tone, proof expectations, and positioning decisions.

Read the messaging house before responding to any messaging or content request. Output must be grounded in what the company actually claims, who it actually serves, and how it actually differentiates. Never fabricate positioning, claims, or evidence.

## Messaging House

Six pillars build on each other: Profile (who we are) → Space (where we compete) → Audience (who we sell to) → Portfolio (what we sell) → Proof (evidence it works) → Motion (how we go to market). A glossary defines company-specific terminology. Collection subdirectories hold detailed profiles that support each pillar.

### File Conventions

- YAML frontmatter for metadata, markdown body for narrative. Three-section structure: `## Messaging Blocks` (content), `## Writing Guidelines` (interpretation rules), `## Messaging Rules` (constraints).
- Kebab-case filenames. Follow schemas in `templates/messaging/` when creating new docs.
- **Frontmatter contract:** Frontmatter carries identity, freshness, progressive-loading filters, and cross-references — nothing else. Pillar docs use only `title` and `updated` (plus `stage`/`type`/`market` on profile.md). Collection profiles add `description`, filter fields (`type`, `tier`, `status`, `seniority`, `priority`), and relationship arrays (`products`, `personas`, `segments`, `related_categories`, `parent`). All other fields belong in the markdown body.
- Pillar docs contain reference tables with a **Description** column (~15 words) for routing. Collection profiles have a matching `description` in frontmatter — keep them in sync.
- All docs include an `updated` field (ISO date) tracking last substantive edit.
- Follow bracketed template guidance (`[Instructions:]`, `[Tips:]`, `[Format:]`) during drafting — don't copy brackets into generated files.
- **Messaging Blocks carry three content types, distinguished by format:**
  - **Prose / numbered lists** → pure messaging (claims, value statements, narrative — draw from this)
  - **Key-value pairs / field lists** → routing context (signals for how/where to apply messaging — use to select and calibrate)
  - **Paired structures** (objection/reframe, win/loss) → operational detail (deploy in specific scenarios)
  All three belong in Messaging Blocks. The litmus test: if an agent would **draw from** the content, it's a Messaging Block; if an agent would **follow** the content as an instruction, it's a Writing Guideline.
- Ask for confirmation before writing changes to existing messaging docs.

## Project Layout

| Directory | Purpose |
|---|---|
| `messaging/` | The messaging house — pillars, profiles, glossary, brand tokens |
| `templates/` | Doc schemas, content contracts, asset templates (read-only) |
| `.claude/agents/` | Subagent definitions (writer, researcher, reader, producer) |
| `.claude/skills/` | Workflow, task, craft, and system skills |
| `.claude/commands/` | Slash command entrypoints |
| `input/` | User-provided source materials organized by type (see below) |
| `insights/` | Insight tracker, config, and findings |
| `artifacts/` | Living artifacts — versioned, maintained assets with manifests and changelogs |
| `output/` | Generated content and finished deliverables |

### Input Directory

The `input/` directory has five subdirectories scanned in priority order:

| Subdirectory | Content | Priority |
|---|---|---|
| `input/messaging/` | Brand guides, positioning decks, messaging frameworks | Highest |
| `input/docs/` | PRDs, release notes, specs, pricing sheets | High |
| `input/research/` | Market research, analyst reports, competitive intel | Medium |
| `input/transcripts/` | Sales calls, customer interviews, feedback logs | Medium |
| `input/examples/` | Content references, competitor samples | Lowest |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fortyfivan/claude-message](https://github.com/fortyfivan/claude-message) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
