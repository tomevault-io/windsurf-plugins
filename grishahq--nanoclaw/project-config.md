---
trigger: always_on
description: Four pillars. Every line of code must serve at least one. If it serves none, delete it.
---

# agents.md - Rules for AI Agents Working on nanoClaw

## Core Principles

Four pillars. Every line of code must serve at least one. If it serves none, delete it.

1. **SECURE** - The user trusts us with shell access to their machine. Every input is hostile until proven safe. Defense in depth: if one layer fails, the next catches it. No security shortcuts, no "we'll fix it later".

2. **LIGHTWEIGHT** - Target: ~3000 lines total. Every file has a line budget (see below). No abstractions without 2+ concrete uses. No wrapper classes that just delegate. If stdlib does it, don't import a package.

3. **FAST** - The user is waiting in Telegram. Target: first response token in <2 seconds for simple queries. Shared connections, async everything, minimal context window usage. Don't send 50 messages of history when 15 is enough. Don't call LLM to extract memories if the message is "thanks".

4. **EFFECTIVE** - The agent must actually solve problems. Bias toward action: call tools, don't describe what tools you could call. Get it done in fewer iterations. A 1-iteration answer is better than a 4-iteration answer if the result is the same.

## Code Size

Target: ~3000 lines total. This is a guideline, not a hard limit per file.

The rule is simple: **every line must earn its place.** If a function can be 10 lines instead of 20 with the same clarity and safety, use 10. But never sacrifice error handling, security checks, or readability to save lines.

If a file feels bloated (300+ lines), that's a signal to split it. But don't split prematurely -- a clean 280-line file is better than three 100-line files with awkward imports.

## Project Language

Everything in English: code, comments, commits, docs, README, variable names, error messages, CLI output. No exceptions.

## Architecture

- The source of truth for architecture is `nanoclaw-spec-v2.md`. Read it before writing any code.
- If you change architecture decisions (add a module, change a dependency, rename something), update the spec file too.
- Keep `ARCHITECTURE.md` in the repo root with a current high-level overview: modules, data flow, dependencies. Update it when structure changes.

## Code Style

- No em dashes (—), en dashes (–), or unicode arrows (→, ←, ↔) anywhere in source code, comments, docstrings, or string literals. Use standard ASCII: `--`, `->`, `<-`, `=>`.
- No smart quotes (" " ' '). Use straight quotes only (" and ').
- No emoji in source code or logs. Emoji are allowed ONLY in user-facing Telegram messages and CLI output where explicitly specified in the spec.
- Type hints on all function signatures. Use `from __future__ import annotations` where needed.
- Docstrings on every public class and function. One-liner is fine for simple ones.
- Max line length: 100 characters.
- Use `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants.
- Imports: stdlib first, then third-party, then local. Separated by blank lines.
- No wildcard imports (`from x import *`).
- Prefer explicit over implicit. No magic. If something is unclear, add a comment.

## Git Commits

- Never mention Claude, AI, LLM, Copilot, or any AI tool in commit messages. Write commits as if a human developer wrote the code.
- Commit message format: `type: short description` where type is one of: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`.
- Examples: `feat: add shell sandbox with three-tier filtering`, `fix: handle timeout in web_fetch`, `docs: update architecture diagram`.
- Keep commits atomic. One logical change per commit. Don't lump unrelated changes together.
- Don't commit generated files, `__pycache__`, `.pyc`, `config.json`, or workspace contents.

## Dependencies

- Minimize dependencies. Every new package needs justification.
- Pin versions in pyproject.toml with minimum bounds (`>=`), not exact pins (`==`).
- No heavy frameworks: no langchain, llamaindex, fastapi, flask, django, react.
- Before adding a dependency, check if stdlib can do it. `sqlite3`, `json`, `asyncio`, `pathlib`, `re`, `datetime`, `importlib` are all stdlib.
- If you add a dependency, add it to pyproject.toml AND mention it in ARCHITECTURE.md.

## Error Handling

- Every `async with`, `await`, and external call must have proper try/except.
- Never catch bare `Exception` and silently pass unless it's explicitly a best-effort operation (like background memory extraction).
- User-facing errors should be clear and actionable: "Web search failed: invalid API key. Run `nanoclaw init` to reconfigure." not "Error occurred."
- Log errors with `logger.error()` including the exception. Don't just print.
- All tools must return a string result, never raise unhandled exceptions to the agent loop.

## Security

- Security code is the most critical part of the project. Be extra careful.
- Never log API keys, tokens, or passwords. Truncate or mask them if needed for debugging.
- Never add API keys or secrets to default config, examples, tests, or README.
- All file paths go through `FileGuard.validate_path()`. No exceptions, no shortcuts.
- All shell commands go through `ShellSandbox`. No direct `subprocess` or `os.system` calls anywhere else.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grishahq/nanoClaw](https://github.com/grishahq/nanoClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
