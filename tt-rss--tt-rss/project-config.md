---
trigger: always_on
description: Tiny Tiny RSS is a web-based RSS/Atom feed reader and aggregator built with PHP (backend) and JavaScript with Dojo Toolkit (frontend). Forked in October 2025 to continue development after original tt-rss.org shutdown.
---

# Tiny Tiny RSS (tt-rss) - AI Coding Agent Instructions

## Project Overview
Tiny Tiny RSS is a web-based RSS/Atom feed reader and aggregator built with PHP (backend) and JavaScript with Dojo Toolkit (frontend). Forked in October 2025 to continue development after original tt-rss.org shutdown.

## CRITICAL: Essential Prerequisites

### Always Examine Actual Source Code
**NEVER make assumptions about code behavior, dependencies, or structure.**

- ❌ **Bad**: "FeedParser probably uses PluginHost for extensibility"
- ✅ **Good**: Read `classes/FeedParser.php` and grep for `PluginHost::` to verify

**Before making any claim about a class**:
1. **Read the actual file** - Use `read_file` tool to examine the source
2. **Check dependencies** - Use `grep -E '(Config::|Prefs::|PluginHost::|Db::)'` to verify what it uses
3. **Verify assumptions** - Don't trust intuition; verify with code

**Examples of dangerous assumptions**:
- Assuming a parser class uses plugins (it might be pure DOM parsing)
- Assuming a utility class needs database (it might be pure logic)
- Assuming behavior based on class name (check actual implementation)

**This applies to**:
- Determining test type (standard vs mocked vs integration)
- Identifying dependencies before refactoring
- Understanding code behavior for bug fixes
- Adding new features to existing classes

### Self-Improving Instructions

**When encountering issues or patterns that could improve future work:**

1. **Identify the gap**: What went wrong? What knowledge was missing?
2. **Root cause analysis**: Why did the issue occur? (e.g., incomplete guidance, missing example, ambiguous wording)
3. **Propose improvement**: Draft specific instruction updates to prevent recurrence
4. **Validate improvement**: Ensure new guidance is clear, actionable, and doesn't contradict existing instructions
5. **Update this file**: Make targeted changes to `.github/copilot-instructions.md`
6. **Provide concise summary in chat**: Summarize changes verbally rather than creating documentation files

**Examples of self-improvements made**:
- **Issue**: FeedParser falsely assumed to use PluginHost
- **Root cause**: Didn't systematically check all methods, relied on intuition
- **Improvement**: Added "Always Examine Actual Source Code" as critical prerequisite with grep workflow
- **Issue**: UrlHelper::url_to_youtube_vid() test missed despite being high-value pure logic
- **Root cause**: Focused only on methods already in test file, no systematic review
- **Improvement**: Added "Completeness Check: Review for Missing Tests" with grep workflow for all public methods
- **Issue**: CSS/Less linting infrastructure created but instructions not updated
- **Root cause**: Forgot to proactively update instructions after completing new code quality workflow
- **Improvement**: Added explicit reminder to update instructions when adding code quality tools, workflows, or discovering project patterns
- **Issue**: Used outdated "CSS3" terminology in documentation
- **Root cause**: Didn't verify terminology against authoritative sources (MDN)
- **Improvement**: Added requirement to check authoritative documentation sources for technical terminology

**Goal**: Continuously improve these instructions based on real-world usage patterns and mistakes.

**CRITICAL: When to Update Instructions Automatically**:
- **New code quality tools added** (linters, formatters, static analyzers) → Update "Code Quality & Testing" section
- **New project conventions discovered** (coding style, architecture patterns) → Add to relevant convention sections
- **Build/workflow changes** (GitHub Actions, Docker, dependencies) → Update relevant workflow sections
- **Deprecation patterns identified** → Add to "Deprecation & Migration" section
- **Testing patterns established** (new test types, bootstrap methods) → Update "Testing & Test Development" section
- **Configuration files created** (`.stylelintrc.json`, `phpstan.neon`, etc.) → Document in relevant sections

**CRITICAL: Verify Terminology Against Authoritative Sources**:
When writing documentation about web technologies, programming languages, or frameworks:
1. **Check official documentation first** - Use authoritative sources for terminology
   - CSS/HTML: https://developer.mozilla.org/en-US/docs/Web/CSS (MDN Web Docs)
   - PHP: https://www.php.net/manual/en/ (Official PHP Manual)
   - JavaScript: https://developer.mozilla.org/en-US/docs/Web/JavaScript (MDN Web Docs)
2. **Avoid outdated terminology** - Don't use deprecated version names or legacy terms
   - ❌ "CSS3", "HTML5" (versioned names are deprecated - use "CSS", "HTML")
   - ❌ "AJAX" (use "fetch API" or "XMLHttpRequest")
   - ❌ "MySQL" for PostgreSQL (verify actual database in use)
3. **Use precise technical terms** - Prefer official specification terminology
   - ✅ "pseudo-elements" not "pseudo elements"
   - ✅ "alpha channel" not "alpha value"
   - ✅ "space-separated syntax" not "new syntax"
4. **When uncertain** - Search the official documentation before documenting
   - Example: Search MDN for "CSS versions" before using "CSS3"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tt-rss/tt-rss](https://github.com/tt-rss/tt-rss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
