---
trigger: always_on
description: `fast` is useful to LLM agents in two different ways:
---

# Using Fast for LLMs and Agents

`fast` is useful to LLM agents in two different ways:

1. As a CLI command the agent can run in a terminal.
2. As an MCP server through `bin/fast-mcp`.

Both help reduce token waste compared with raw text grep. The right integration depends on what the host can support.

## Why use `fast` over `grep`

- **Precision**: You can search for syntactic constructs (e.g., classes, method definitions, specific method calls) rather than just substrings.
- **Context Preservation**: `fast` inherently understands the bounds of a method or block. It will print the entire body of the AST node matched, not just the single line containing the keyword.
- **Token Efficiency**: Get only the function or class you want, instead of 100 lines of regex false positives.
- **File and Line Locality**: Outcomes are directly prefixed with the file and line number by default (`# file/path.rb:123`), making it trivial to know exactly where the code is located for further editing or patching.
- **Safer Rewrites**: Fast validates rewritten Ruby before returning it or writing it to disk. Invalid replacements fail with an error instead of silently producing broken code.

## CLI or MCP

Choose CLI when:

- The host can run shell commands but cannot register MCP servers.
- You want the smallest possible setup and best portability.
- The agent is already comfortable parsing terminal output.

Choose MCP when:

- The host supports MCP registration and tool calling.
- You want typed arguments and structured JSON results.
- You want the model to call targeted tools like `ruby_method_source` instead of constructing shell commands.

For Codex-style terminal agents, the CLI is immediately useful and often enough. For IDE agents and multi-tool hosts, MCP is usually the better interface because it removes output parsing and makes tool selection explicit.

## Trust boundary and security

`fast` is a local developer tool. The CLI and MCP server should only be used with trusted local hosts, trusted repositories, and trusted prompts.

This matters more for MCP than for plain CLI usage:

- search tools can read and return source from files you point them at
- `rewrite_ruby_file` can modify files on disk
- `run_fast_experiment` can execute Ruby and shell commands as the current user

That means `bin/fast-mcp` is not appropriate to expose to untrusted remote clients, shared multi-tenant environments, or any host you would not trust with normal shell access to your machine.

If you only need read-only exploration and do not trust the host with file writes or command execution, prefer the CLI and avoid registering the MCP server.

## Essential CLI flags for agents

To maximize reliability and reduce context noise, use these flags when invoking `fast` from the command line:

- `--no-color`: **CRITICAL**. Always use this flag to strip ANSI escape codes formatting out of the output. TTY color codes consume unnecessary tokens and break markdown parsing.
- `--headless`: Omits the `# filename.rb:line` header if you only want the raw code snippet and don't care about the location.
- `--bodyless`: Omits the code block body and only shows the matched headers (useful for finding *where* something is without reading *what* it is).
- `--ast`: Prints the S-expression representation of the matching nodes. Outstanding when you need to understand the internal AST structure of a complex ruby construct to construct more advanced `fast` queries or RuboCop node patterns.

## Use `.summary` for first-pass reconnaissance

`fast .summary file.rb` is a high-leverage shortcut for agents entering a large file. Instead of reading the full body first, it prints a compact structural outline:

- class and module nesting
- constants
- mixins
- relationships such as `has_many` and `belongs_to`
- attributes such as `attr_reader`
- scopes, hooks, and validations
- method signatures grouped by visibility
- macro-heavy sections that would otherwise waste tokens

This is especially useful before deciding whether to fetch full method bodies through `fast`, MCP tools, or ordinary file reads.

`.summary` also supports `-l` / `--level`:

- `-l 1`: only class/module inventory
- `-l 2`: inventory plus signals such as constants, hooks, validations, scopes, and macros
- `-l 3`: inventory, signals, and method signatures

Example:

```bash
fast .summary lib/fast/mcp_server.rb -l 2 --no-color
```

## Use `.scan` for multi-file triage

`fast .scan path/to/dir --no-color` extends the same idea across many files. It classifies files into broad groups and prints a bounded per-file outline without dumping full bodies.

The scanner is designed to help agents avoid rabbit holes during repo exploration:

- group files into models, controllers, services, jobs, mailers, libraries, and other
- show one short headline per structural entry
- surface only the most useful signals such as hooks, validations, relationships, mixins, and macros
- list a capped set of public and private method names
- avoid printing method bodies, large constants, or implementation details by default

This makes `.scan` a better first move than reading a whole directory tree when the task is still about classification and narrowing scope.

`.scan` also supports `-l` / `--level`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonatas/fast](https://github.com/jonatas/fast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
