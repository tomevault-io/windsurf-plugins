---
trigger: always_on
description: This repository contains an Obsidian plugin for semantic search using local
---

# Development guide

This repository contains an Obsidian plugin for semantic search using local
embeddings.

## Prerequisites & installation

See [README.md](./README.md#requirements) for prerequisites and installation
instructions.

## Build

```bash
npm run build                 # Production build (without benchmark code)
npm run build:with-benchmark  # Development build (includes benchmark code)
```

Benchmark code is excluded from production builds to keep the plugin small. Use
`build:with-benchmark` when you need to run retrieval or RAG benchmarks.

## Code quality checks

```bash
npm run check         # Comprehensive check: format + lint + strict type checking
npm run check:format  # Check formatting -- included in `npm run check`
npm run check:lint    # Check for ESLint errors -- included in `npm run check`
npm run check:svelte  # Svelte check -- included in `npm run check`
npm run check:tsc     # Strict type checking (no `skipLibCheck`) -- included in `npm run check`
```

For Python files in `retrieval-bench/scripts/`:

```bash
uv run basedpyright   # Type checking for Python code
uv run ruff check .   # Linting for Python code (checks formatting, line length, etc.)
```

## Code quality fixes

```bash
npm run fix           # Auto-format + auto-fix linting (combined)
npm run fix:format    # Auto-format code with Prettier -- included in `npm run fix`
npm run fix:lint      # Auto-fix ESLint errors -- included in `npm run fix`
```

## Testing

This project uses Vitest as the testing framework. Test files are located in
`src/` directory alongside the source files (e.g., `src/chunker.test.ts` for
`src/chunker.ts`).

```bash
npm run test        # Run all tests once
npm run test:watch  # Run tests in watch mode
npm run test:ui     # Run tests with interactive browser UI
```

## Development

### General guidelines

- **Make sure to run the code quality checks after making changes**
  - During development: Run `npm run build` to verify compilation succeeds
  - Before finalizing/committing:
    1. Run `npm run test` to verify all tests pass
    2. Run `npm run build` to verify that the build completes without issues
    3. Run `npm run check` for comprehensive validation (format check, ESLint
       with 0 warnings, and strict TypeScript type checking without
       `skipLibCheck`)
    4. Run `npm run fix` to adjust code style if needed
  - For Python files in `retrieval-bench/scripts/`: Run `uv run basedpyright`
    and `uv run ruff check .`
- Keep the plugin small. Avoid large dependencies. Prefer browser-compatible
  packages.
- Avoid Node/Electron APIs where possible.
- **Place temporary and experimental scripts in `/experiments` directory**: All
  one-off scripts, prototypes, or experimental code should be placed under the
  `/experiments` directory. This keeps the main codebase clean and makes it
  clear which code is production vs. experimental.

### Coding style

**[!IMPORTANT]: ALWAYS REMEMBER WITH HIGH PRIORITY**

- All code, documentation and comments should be written in English
  - If instructions are given in a language other than English, you may respond
    in that language
  - But code/documentation/comments must be written in English unless explicitly
    requested in the instructions
- **Do not leave unnecessary comments in code**
  - Instead prefer self-documenting code with clear variable, function names,
    and data/control flows
- **When writing comments or documentation, avoid excessive decoration**. For
  example, avoid scattering emojis or overusing `**` bold formatting. Use these
  only where truly necessary.
- Use backticks for code references: When writing comments, commit messages, or
  documentation, wrap code-related terms in backticks (e.g., `functionName`,
  `variableName`, `file.ts`) to distinguish them from regular text.
- When writing Markdown text, use _2 whitespaces_ for indentation and try to
  keep the maximum line length under _80 characters_.
  - Additionally, prioritize simple text style and limit unnecessary decorations
    (e.g. `**`) to only truly necessary locations. This is a style that should
    generally be aimed for, but pay particular attention when writing Markdown.
  - Headers should use sentence case (only the first word capitalized), not
    title case. For example:
    - Good: `## Conclusion and alternative approaches`
    - Bad: `## Conclusion And Alternative Approaches`
- Commit messages:
  - Do not include the "Generated with
    [Claude Code](https://claude.com/claude-code)" footer in commit messages for
    this project. Keep commit messages focused and concise.
  - When writing commit messages, follow the format "component: Brief summary"
    for the title. In the body of the commit message, provide a brief prose
    summary of the purpose of the changes made. Use backticks for code elements
    (function names, variables, file paths, etc.) to improve readability. Also,
    ensure that the maximum line length never exceeds 72 characters. When
    referencing external GitHub PRs or issues, use proper GitHub interlinking
    format (e.g., "owner/repo#123" for PRs/issues).
- Keep `main.ts` minimal: Focus only on plugin lifecycle (onload, onunload,
  addCommand calls). Delegate all feature logic to separate modules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aviatesk/obsidian-sonar](https://github.com/aviatesk/obsidian-sonar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
