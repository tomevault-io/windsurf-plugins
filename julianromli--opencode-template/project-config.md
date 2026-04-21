---
trigger: always_on
description: - **Project-Specific Documentation Priority:** When working on any project, FIRST check for project-specific documentation in this priority order:
---

# Global Agent Rules

- **Project-Specific Documentation Priority:** When working on any project, FIRST check for project-specific documentation in this priority order:
  1. Project root `AGENTS.md` (if exists)
  2. `.next-docs/` directory (for Next.js projects, created by `npx @next/codemod@canary agents-md`)
  3. `docs/` folder for project-specific guides
  4. Then fallback to this global AGENTS.md

- **Frontend Developer Agent - MANDATORY Skill Invocation:** The `frontend-developer` subagent MUST ALWAYS call `use_skill("frontend-design")` as its VERY FIRST action before doing ANY work. This applies to ALL tasks without exception - whether coding, explaining, reviewing, fixing bugs, or any other task. NO EXCEPTIONS. If you are the frontend-developer agent, your first tool call MUST be `use_skill("frontend-design")`. Failure to do so is a critical violation.

- **Environment Files Safety:** ALWAYS use the `fs_read` and `fs_write` tools when accessing or modifying `.env`, `.env.local`, `.env.example`, or any other sensitive environment configuration files. NEVER use the basic `read` or `edit` tools for these files to avoid permission issues and ensure proper handling of sensitive data.

- Warp Grep: warp-grep is a subagent that takes in a search string and tries to find relevant context. Best practice is to use it at the beginning of codebase explorations to fast track finding relevant files/lines. Do not use it to pin point keywords, but use it for broader semantic queries. "Find the XYZ flow", "How does XYZ work", "Where is XYZ handled?", "Where is <error message> coming from?"

- **Mgrep (MANDATORY for Search):** `mgrep` COMPLETELY REPLACES traditional `grep` for code search. When you need to "grep" or "search" the codebase, you MUST use the `mgrep` tool, NOT the bash `grep` command. Use `mgrep` for both semantic queries ("where is auth?") and specific keywords. **DO NOT** use `grep` via the `bash` tool unless explicitly performing complex regex operations that `mgrep` cannot handle. If the user says "grep", they mean "use the mgrep tool". Before searching a new project, ensure `mgrep watch` is running in that project's root to index files. Mgrep supports PDFs, images, and code.

- Always use Context7 MCP when I need library/API documentation, code generation, setup or configuration steps without me having to explicitly ask.

- Firecrawl MCP: firecrawl is the primary web scraping and search tool. Use `firecrawl_search` for web searches (prefer over WebSearch), `firecrawl_scrape` for single page content, `firecrawl_map` to discover URLs on a site before scraping. Best practice is to search first WITHOUT scrapeOptions to get URLs, then scrape the relevant results separately. Add `maxAge` parameter for 500% faster cached responses. Use search operators like `site:example .com`, `"exact phrase"`, `-exclude`.

- Context7 MCP: context7 provides up-to-date library documentation. Always call `resolve-library-id` first to get the library ID (e.g., "/prisma/prisma"), then call `query-docs` with that ID. Be specific in queries—"How to set up JWT auth in Express" not just "auth". Do not call more than 3 times per question; use best result if not found after 3 attempts.

- Verify & Iterate: After any implementation, setup, or code change, always verify it works by running the app, tests, or build. If it fails, errors, or exits unexpectedly, debug and fix immediately—do not move on until it works. Test behavior, not implementation. When fixing bugs, reproduce first, then fix, then verify the fix.

# Web Interface Guidelines

Concise rules for building accessible, fast, delightful UIs. Use MUST/SHOULD/NEVER to guide decisions.

## Interactions

### Keyboard

- MUST: Full keyboard support per [WAI-ARIA APG](https://www.w3.org/WAI/ARIA/apg/patterns/)
- MUST: Visible focus rings (`:focus-visible`; group with `:focus-within`)
- MUST: Manage focus (trap, move, return) per APG patterns
- NEVER: `outline: none` without visible focus replacement

### Targets & Input

- MUST: Hit target ≥24px (mobile ≥44px); if visual <24px, expand hit area
- MUST: Mobile `<input>` font-size ≥16px to prevent iOS zoom
- NEVER: Disable browser zoom (`user-scalable=no`, `maximum-scale=1`)
- MUST: `touch-action: manipulation` to prevent double-tap zoom
- SHOULD: Set `-webkit-tap-highlight-color` to match design

### Forms

- MUST: Hydration-safe inputs (no lost focus/value)
- NEVER: Block paste in `<input>`/`<textarea>`
- MUST: Loading buttons show spinner and keep original label
- MUST: Enter submits focused input; in `<textarea>`, ⌘/Ctrl+Enter submits
- MUST: Keep submit enabled until request starts; then disable with spinner
- MUST: Accept free text, validate after—don't block typing
- MUST: Allow incomplete form submission to surface validation
- MUST: Errors inline next to fields; on submit, focus first error
- MUST: `autocomplete` + meaningful `name`; correct `type` and `inputmode`
- SHOULD: Disable spellcheck for emails/codes/usernames
- SHOULD: Placeholders end with `…` and show example pattern
- MUST: Warn on unsaved changes before navigation
- MUST: Compatible with password managers & 2FA; allow pasting codes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [julianromli/opencode-template](https://github.com/julianromli/opencode-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
