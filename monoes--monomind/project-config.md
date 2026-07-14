---
trigger: always_on
description: > **Monomind v2.0.0** — Packages: `monomind@2.0.0` (umbrella), `@monoes/monomindcli@2.0.0` (CLI), `@monoes/monograph@1.4.0` (knowledge graph)
---

# Claude Code Configuration - Monomind v2

> **Monomind v2.0.0** — Packages: `monomind@2.0.0` (umbrella), `@monoes/monomindcli@2.0.0` (CLI), `@monoes/monograph@1.4.0` (knowledge graph)

## Behavioral Rules (Always Enforced)

- For swarm/hive-mind mode selection, use `/mastermind` — it presents all topologies and gives a concrete recommendation. Do NOT auto-prompt for swarm mode.
- For ANY UI testing, browser automation, or web navigation request: ALWAYS invoke `Skill("agent-browser-testing")` FIRST — no exceptions. Uses native `monomind browse` CDP client — no external binary needed.
- NEVER use `mcp__claude-in-chrome__*`, `mcp__plugin_playwright__*`, `mcp__playwright__*`, Playwright, Puppeteer, Selenium, or any external browser tool for web browsing. ALWAYS use `npx monomind browse`. This rule has no exceptions — not even "just this once".
- For ANY web animation, motion graphics, or animation request: ALWAYS invoke `Skill("monomotion")` FIRST — no exceptions. This includes: "animate this", "add animation", "create an animation", "motion graphics", "animated intro/outro", "text animation", "scroll animation", "GSAP".
- For ANY frontend design, UI improvement, design critique, design system, brand identity, UX research, visual storytelling, image generation for design, component systems, or CSS architecture task: ALWAYS invoke `Skill("monodesign")` FIRST — no exceptions. This is the ONLY design agent — there are no separate UI Designer, UX Architect, UX Researcher, Brand Guardian, Visual Storyteller, Whimsy Injector, Image Prompt Engineer, or Inclusive Visuals agents anymore. All design intelligence is in monodesign. This includes: "design this", "redesign", "improve the UI", "add polish", "make it look better", "audit the design", "critique the UI", "fix the layout", "colorize", "typeset", "design system", "design tokens", "antipattern", "brand identity", "brand strategy", "ux research", "user research", "usability test", "persona", "component system", "css architecture", "theme toggle", "dark mode", "image prompt", "hero image", "generate image", "whimsy", "delight", "visual narrative", "inclusive design".
- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (\*.md) or README files unless explicitly requested
- NEVER save working files, text/mds, or tests to the root folder
- Never continuously check status after spawning a swarm — wait for results
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or .env files
- ALWAYS call `mcp__monomind__monograph_query` BEFORE running grep/rg/find via Bash for code exploration — only fall back to Bash grep if monograph returns 0 results or the DB does not exist
- When starting any task that touches 3+ files: call `mcp__monomind__monograph_suggest` first to get relevant nodes ranked by task relevance
- **Crash reporting**: monomind, mono-agent, monotask, and mono-clip each auto-report uncaught crashes as GitHub issues on their own repo (on by default — `monomind crash-reporting disable` to opt out; see `packages/@monomind/cli/src/services/crash-reporter.ts`). This only covers hard crashes, not everyday friction — so when a user is stuck on something that ISN'T a crash (a confusing error message, a workflow that doesn't behave as documented, something in these four tools that seems broken or inconsistent), **suggest opening a GitHub issue** rather than filing one yourself. One line is enough: name the repo (`monoes/monomind`, `monoes/mono-agent`, `monoes/monotask`, or `monoes/mono-clip`, whichever is actually implicated) and ask if they'd like you to open it (via `gh issue create`) or if they'd rather do it themselves. Don't do this for run-of-the-mill usage questions you can just answer — only when something is genuinely unresolved, contradictory, or looks like a real bug in one of these four tools.

## File Organization

- NEVER save to root folder — use the directories below
- Use `/src` for source code files
- Use `/tests` for test files
- Use `/docs` for documentation and markdown files
- Use `/config` for configuration files
- Use `/scripts` for utility scripts
- Use `/examples` for example code

## Project Architecture

- Follow Domain-Driven Design with bounded contexts
- Keep files under 500 lines
- Use typed interfaces for all public APIs
- Prefer TDD London School (mock-first) for new code
- Use event sourcing for state changes
- Ensure input validation at system boundaries

### Key Packages

| Package               | Path                            | Purpose                                |
| --------------------- | ------------------------------- | -------------------------------------- |
| `@monomind/cli`      | `packages/@monomind/cli/`      | CLI entry point (31 commands)          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [monoes/monomind](https://github.com/monoes/monomind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
