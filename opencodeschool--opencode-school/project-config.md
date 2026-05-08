---
trigger: always_on
description: See [README.md](README.md) for setup instructions and [CONTRIBUTING.md](CONTRIBUTING.md) for how to contribute.
---

See [README.md](README.md) for setup instructions and [CONTRIBUTING.md](CONTRIBUTING.md) for how to contribute.

## Project overview

OpenCode School (opencode.school) is an interactive course that teaches people how to use OpenCode. Built with Astro 6, deployed to Cloudflare Workers, with student progress tracked via Cloudflare KV.

OpenCode is a general-purpose AI agent — not exclusively a coding tool. While it has strong capabilities for software development tasks, it can help with writing, research, data analysis, and many other non-coding tasks. When writing lesson content or agent instructions, avoid framing OpenCode as coding-only.

## OpenCode Desktop

Students in this course primarily use OpenCode Desktop — a native Electron-based app with a graphical interface (GUI) where students interact using mouse and keyboard. It is different from the OpenCode TUI (terminal interface) and does not support TUI-specific features like the `/connect` slash command.

Install methods:
- macOS: `brew install --cask opencode-desktop` or download .dmg from opencode.ai/download
- Windows: download installer from opencode.ai/download (WSL recommended)
- Linux: download .deb or .rpm from opencode.ai/download

Several lessons require students to quit and reopen Desktop for changes to take effect (e.g. after editing config, adding MCP servers, installing skills, or installing plugins). When writing lesson content or agent instructions that involve restarting, refer to "OpenCode Desktop" specifically, not just "OpenCode".

## Stack

- [Astro](https://astro.build) — static site framework
- [Cloudflare Workers](https://workers.cloudflare.com) — hosting
- [Cloudflare KV](https://developers.cloudflare.com/kv/) — student progress storage (binding: `PROGRESS`)
- [Cloudflare R2](https://developers.cloudflare.com/r2/) — video asset storage (bucket: `opencodeschool-assets`, binding: `ASSETS_BUCKET`, served at `https://assets.opencode.school`)

## Agent-facing files

This project serves two audiences: human students (via the website) and AI agents (via the API and discovery files). Three files work together to make the API discoverable:

- `src/pages/llms.txt.ts` — dynamic Astro route that serves a plain-text overview of the site and API for LLM agents at `/llms.txt`. Points to the OpenAPI spec and gives agents their operating instructions. This is the first thing an agent reads when visiting the site. Do not edit `public/llms.txt` directly — it is not used at runtime.
- `src/pages/api/openapi.json.ts` — dynamic Astro route that serves the OpenAPI 3.1 spec at `/api/openapi.json`, describing all API endpoints, request/response schemas, and examples.
- `src/pages/api/lessons/index.ts` and `src/pages/api/lessons/[slug].ts` — the actual API endpoints that serve lesson content and agent instructions as JSON.

When adding or removing API endpoints or lessons, update both of these in the same commit:

1. Add/update the route handler in `src/pages/api/`
2. Update `src/pages/api/openapi.json.ts` with the new endpoint schema

Agent-facing URLs (`/llms.txt`, `/api/openapi.json`, `/api/instructions/{studentId}`, lesson prompts) must use the current request or page origin, not a hardcoded domain. The lessons API replaces the `{origin}` placeholder in `agentInstructions` with the request origin at serve time, so use `{origin}` in MDX frontmatter when an agent-facing URL needs the site origin.

## Lesson content

Each lesson is an MDX file in `src/content/lessons/`. The frontmatter schema (defined in `src/content.config.ts`) includes:

- `title`, `slug`, `description`, `order` — standard metadata
- `agentInstructions` (required) — describes what "done" looks like and how to verify it; used by agents to evaluate and mark completion

When editing lessons, keep `agentInstructions` accurate. It should describe both the verifiable end state and the steps an agent should take to confirm it. Always use YAML literal block scalars (`|`) for `agentInstructions`, never quoted strings. This keeps instructions readable, produces line-level diffs, and allows paragraph breaks between logical sections.

## Lesson authoring guidelines

### Quiz format

Lessons with `quiz: true` in their frontmatter use a teach → quiz → verify flow. The set of quiz lessons may grow over time. The shared quiz boilerplate is defined in `src/lib/quiz-instructions.ts` and injected by the API layer at runtime — do not copy it into individual MDX files.

For these lessons, `agentInstructions` in the MDX should contain only:

1. A list of four topics to teach and quiz (numbered, one sentence each)
2. A verification step describing what file to read and what to confirm (omit for lessons with no file-system check)

Keep the instructions concise — the API appends the quiz boilerplate automatically.

### Adding new lessons

The numeric prefix in filenames (e.g. `01-`, `02-`) is for human readability only. Lesson ordering is controlled by the `order` frontmatter field. Routing and the API use `slug`, not the filename or `order`. When renaming or adding lessons, keep filenames, `order` values, and `slug` values in sync, but the filename prefix has no functional effect.

When adding a new OpenCode-based lesson with sufficient content:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opencodeschool/opencode.school](https://github.com/opencodeschool/opencode.school) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
