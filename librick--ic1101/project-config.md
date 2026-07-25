---
trigger: always_on
description: This document is a style guide for Python code in this project. It is read by LLMs generating or modifying code in the repo.
---

# Style Guide

## Purpose

This document is a style guide for Python code in this project. It is read by LLMs generating or modifying code in the repo.

The rules here are not a complete Python style guide; universally-followed conventions (PEP 8, snake_case, module-level loggers, standard level semantics) are assumed and not restated. What follows is specifically the set of conventions that LLMs don't follow by default; rules that override common LLM defaults or encode project-specific decisions. Prefer automated tooling to style guides whenever possible; rules here exist because tooling can't enforce them.

Apply these rules when writing new code and when modifying existing code. When a rule and existing code conflict, update the existing code to match the rule unless doing so is out of scope for the current task. Limit changes to what's necessary to apply the rule; don't rewrite adjacent code beyond what was asked.

If a rule conflicts with an explicit instruction in the current prompt, the prompt wins for that task, but flag the conflict in the response so the style guide can be updated if needed.

## Tools

Tools that run against this codebase. Don't restate conventions these already enforce.

- uv
- pre-commit
- ruff-check
- ruff-format
- pyright
- typos
- mdformat
- pytest

## Style

- Don't use em dashes anywhere; use semicolons, commas, or two sentences
- Multi-line strings require explicit `+` between fragments; implicit adjacent-literal concatenation is banned by the linter

## Naming

- Put more generic terms nearer the start of an identifier, more specific as you move right.
  - Files: `parser_yaml.py`, `parser_json.py`
  - Classes: `ParserYaml`, `ParserJson`
  - Functions: `parse_yaml`, `parse_json`
- Don't uppercase abbreviations, even common ones (prefer ParserJson to ParserJSON)

## Imports

- Imports go at the top of the file; don't inline imports inside functions to avoid circular dependencies; fix the dependency structure instead

## CLI tool invocation

- When invoking CLI tools from Python, prefer long-form subcommands over short aliases for readability (e.g., `apktool install-framework` over `apktool if`)

## Comments

- Don't use banner comments (ASCII borders, section delineators)
- When refactoring, don't explain the reason for the refactor in the comment; the comment describes the code, not its history

## Logging

- Don't use f-strings in log calls; f-strings evaluate at call time even when the log level is disabled. Use `%s`-style format strings with separate arguments: `logger.info("found %d files in %s", n, path)`
- Logging is for observability; exceptions are for control flow. Don't log and raise for the same event; the caller's `logger.exception` catches the detail
- Log at the layer that has the relevant context; a deep utility function usually shouldn't log, because it doesn't know whether its caller is in a noisy batch job or a quiet script
- Library code may call `logger.info` / `logger.warning` / etc. on its module-level logger, but must not configure handlers, call `logging.basicConfig`, or otherwise install logging infrastructure; that's the consuming application's job
- When logging about a file, directory, or path, include the path in the log message; "extracted archive" is useless for debugging, "extracted archive to /path/to/dest" is not
- Log messages should include the specific values they're about: paths, URLs, IDs, counts, command args. A message that says only what happened without saying what it happened to is rarely useful for debugging

## Log message format

- Don't capitalize the first character of log messages unless there's a compelling context-sensitive reason to do so
- Values (paths, IDs, counts) appear unquoted in context unless quoting prevents ambiguity
- Use a colon for labeled values: `exit code: 1`, `output directory: /tmp/foo`
- One sentence per log line; split multi-part information into multiple log calls
- Include units on numbers when the unit isn't obvious (`0.3s`, `1024 bytes`)
- Error messages describe the condition as a statement, not as a command to the reader

## Docstrings

- Use Google-style docstring sections when they add information; omit them when they would only restate the signature
- Only `Args:`, `Returns:`/`Yields:`, and `Raises:` sections are permitted; other Google-style sections should be written as prose in the body of the docstring
- Do NOT use reStructuredText/Sphinx field syntax (`:param:`, `:returns:`, `:raises:`)
- Do NOT use NumPy-style underlined headings
- A docstring without any sections is fine and often preferable; don't add sections for their own sake
- Within a section, document only the entries that need explanation; partial coverage is better than padded coverage
- Keep docstrings short; if you find yourself writing a paragraph per parameter, the docstring is doing too much
- Don't treat `Raises:` as exhaustive; only list exceptions callers are meant to catch programmatically
- Don't list a single domain exception in `Raises:` if the exception class is defined at module scope nearby
- For parameters that map directly to external tool flags or APIs, lead with the mechanical mapping before the semantic explanation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [librick/ic1101](https://github.com/librick/ic1101) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
